Thanks for laying these out. I'd go with **C**.

Ruling out the others first:

- **D** doesn't fix the issue. As you noted, contexts copied into background tasks keep the map alive.
- **B** goes against how NiceGUI treats late calls on deleted elements, and the assert disappears under `-O`.
- **E** looks like the right 4.0 answer, but it changes behaviour per task and needs its own concurrency tests, so I'd leave it for a follow-up PR.

Between A and C, I'd pick C for two reasons:

1. **A still leaks, reproducibly.** If a timer reads `m.marker` after the client is gone, the class variable keeps that map pinned after the timer lets go, together with everything its layers reference, until the next `ui.leaflet` is used anywhere in the process. With a large GeoJSON layer that could possibly be hundreds of MB, but I haven't got around to checking if that can be the case. The test below fails on f1e00750 (the contributor's commit) and passes with C.
2. **C is small, and a fix like this is a nice place to meet weak references.** They're something you eventually stumble into once you get far enough into the weeds, not something anyone needs to know to contribute. [contributor], I added a short note below in case it's useful.

<details><summary>MRE: late access after client deletion</summary>

```python
async def test_late_marker_then_release(user: User):
    held = []  # stands in for a timer or handler holding the map
    maps: weakref.WeakSet = weakref.WeakSet()

    @ui.page('/')
    def page():
        m = ui.leaflet(center=(51.5, -0.09))
        held.append(m)
        maps.add(m)

    await user.open('/')
    user.client.delete()
    held[0].marker(latlng=(51.5, -0.09))  # late access after the client is gone
    held.clear()  # the timer finishes and drops its reference
    gc.collect()
    assert len(maps) == 0
```

| Tree | Late marker, then release | Freed after next `ui.leaflet` |
|---|---|---|
| f1e00750 (A) | fails, map still alive | passes |
| f1e00750 + weakref (C) | passes | passes |

With C, the existing `test_leaflet_is_collected_after_client_deletion` also passes without the clearing in `_handle_delete`. The same check for `ui.scene` is left as an exercise to the reader.

</details>

---

### A short note on weak references

Python really cares about **references**, because:

- Every variable, list entry or attribute that points to an object is a reference to it.
- Python keeps an object in memory as long as at least one reference to it exists.
- Once the last reference is gone, Python frees that memory. That's how it cleans up without you freeing anything by hand.
- The catch: one leftover reference keeps the object in memory, along with everything *it* refers to. People say the reference "pins" the object.

That's what happened here. `Layer.current_leaflet` is a class attribute, so it lives as long as the program does, and it kept pointing at the last map after its browser tab was closed.

A **weak reference** points to an object without counting as a reference:

- It doesn't keep the object alive.
- You call it to get the object back.
- You get `None` once the object has been freed.

```python
import weakref

class Map: pass

m = Map()
r = weakref.ref(m)
r()        # the Map, because the variable m still refers to it
del m      # the last normal reference is gone, so Python frees the Map
r()        # None
```

Why it fits here:

- `Layer.current_leaflet` only needs the map for a moment. Writing `m.marker(...)` first saves `m` in `Layer.current_leaflet`, then creates the marker, which picks the map up from there.
- At that moment your code is holding the map in `m`, so the weak reference always finds it. The `assert` only fails if no map was ever saved, exactly as before.
- After that, the weak reference doesn't keep the map in memory. Once NiceGUI deletes the client and nothing else refers to the map, Python frees it.

Your fix found where the leak lives: that one variable. The weak reference takes it one step further, so the clearing in `_handle_delete` isn't needed any more, and a late `m.marker` after deletion can't pin the map again (the case in the test above):

```python
# leaflet.py
Layer.current_leaflet = weakref.ref(self)

# leaflet_layer.py
leaflet = Layer.current_leaflet() if Layer.current_leaflet else None  # None if nothing was ever saved
assert leaflet is not None
self.leaflet = leaflet
```

`ui.scene` has the same pattern with `Object3D.current_scene`, so the same change applies there.

---

*For your agent: adding "high signal, low noise" to the prompt is a cheap way to get fewer options to choose from.*[^1] [※](https://zauberzeug.github.io/colophon/#m=claude-opus-5&t=Wording%2C%20the%20MRE%20and%20the%20weakref%20note%20from%20the%20model%3B%20the%20choice%20of%20option%2C%20the%20framing%20and%20the%20edits%20from%20the%20author.&a=claude-code "claude-opus-5: Wording, the MRE and the weakref note from the model; the choice of option, the framing and the edits from the author.")

[^1]: I mentioned this on Discord a while back. Putting it here too, since it's easy to lose.

