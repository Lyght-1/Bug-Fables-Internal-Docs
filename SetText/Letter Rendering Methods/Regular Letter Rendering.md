# Regular Letter Rendering

This letter rendering method is the default one and it is generally the most stable because it was implemented from the start. Every letter is rendered with this method unless [Single](../Commands/Individual%20commands/Single.md) is explicitly toggled on. It involves rendering each letter as individual TextMesh objects (called letter slots) with a maximum of 500 letters that can be rendered using this method alone.

Its main advantage is its compatibility: it works with every features because having each letter be its own TextMesh allows fine control over its meshes. This allows several font effects and other rendering [Commands](../Commands/Commands.md) to work correctly while [Single Letter Rendering](Single%20Letter%20Rendering.md) would be incompatible with them.

Its main problem is its inefficiency because it renders each letter in its own  object. This creates a ton of allocations/garbage collection every time rendering needs to happen with several letters on the screen which can lead to poor performance especially on embedded consoles such as the Nintendo Switch. Additionally, it limits more the amount of text that can be shown on screen at once because it consumes much more letter slots than the 500 available.

## Rendering process

* In [Dialogue mode](../Dialogue%20mode.md) mode, sets [tailtarget](../Notable%20local%20variable/tailtarget.md) to be talking
* If in [Dialogue mode](../Dialogue%20mode.md) mode and the current letter isn't the last one, play the current bleep sound with the current pitch and volume settings and accumulate the letter in the [Backtracking](../Related%20Systems/Backtracking.md) system.
* Get the first free letter slot and store it in letter
* If a slot was available
  * Set the appropriate layer of the letter (this changes the camera that renders the letter):
    * 15 (3DUI) if [Triui](../Commands/Individual%20commands/Triui.md) is enabled (will render using the 3DGUI camera)
    * 5 (UI) if tridimensional's parameter is false (will render using the GUICamera, this is the default)
    * 0 (Default) if tridimensional's parameter is true (will render using the main camera of the game)
  * Set the letter's tag to `Letter`
  * Reserve the letter slot to be the current char, with a font of [fonttype](../fonttype.md), with the parent being the [textholder](../Notable%20local%20variable/textholder.md) at (current offset, current line - 0.1, 0.0) + (0.0, -0.1), using the current text [Color](../Commands/Individual%20commands/Color.md), a sort of [Sort](../Commands/Individual%20commands/Sort.md) and a size of (size.x, size.y, 1.0) * 0.07
  * If we are using [Dropshadow](../Commands/Individual%20commands/Dropshadow.md)
    * Sets the `ds` field to the first free slot and set its layer to the letter's layer
    * Reserve the `ds` slot to be letter's text, with a font of [fonttype](../fonttype.md), with the parent being [textholder](../Notable%20local%20variable/textholder.md) at the letter's localPosition + the [Dropshadow](../Commands/Individual%20commands/Dropshadow.md) offset, using a half transparent black color if [Fadeletter](../Commands/Individual%20commands/Fadeletter.md) is disabled and clear color otherwise, a sort of the current [Sort](../Commands/Individual%20commands/Sort.md) and a size of the letter's scale.
    * If [Fadeletter](../Commands/Individual%20commands/Fadeletter.md) is true, starts a fade of `ds`'s MeshRender to half transparent over the course of 200 frames
    * Increment the letter's MeshRenderer's sortingOrder so it is rendered above the `ds` one
  * If any of the font effects is enabled, add a `FontEffects` to letter with the current effects desired (also sets `superglitch`)
  * Add GetLetterOffset of the current character with font [fonttype](../fonttype.md) and size.x to current offset
* Sets `maxlength` to the current offset if it is larger than the current value
* If [Center](../Commands/Individual%20commands/Center.md) is enabled, sets the [textholder](../Notable%20local%20variable/textholder.md)'s localPosition to (position.x - `maxlenght` / 2.0, position.y, position.z)
* Decide whether to do the letter yield. The conditions are:
  * We are in a [Minibubble](../Commands/Individual%20commands/Minibubble.md)'s inner call in which case, it will always do the yield
  * If not, we need to be in [Dialogue mode](../Dialogue%20mode.md) 
  * If we are, the current [Speed](../Commands/Individual%20commands/Speed.md) needs to be above 0 
  * If it is, this is dependant on the [Text advance](../Related%20Systems/Text%20advance.md): if `skiptext` isn't active OR  [Noskip](../Commands/Individual%20commands/Noskip.md) is active, then the yield will happen
* If we decided to yield, do it for [Speed](../Commands/Individual%20commands/Speed.md) seconds
* An additional yield could be done of 0.15 seconds except on some conditions using the same base condition than the general yield (with the exception that this ignores [Noskip](../Commands/Individual%20commands/Noskip.md)). The additional conditions to do this yield are that the current letter passed to char.IsPunctuation is true AND it's not among `|`, `)`, `¿`, `¡`, `'`, `/`, `¿`, `)` or `¡` AND It's not among the last 2 letters
