# Research All Technologies In Sandbox World

I booted up a fresh single-player world in _Sandbox_ mode so that I could
experiment with some circuitry. As the map is initialized, Factorio asks you a
few questions about how it should be set up. This included a question about
whether all technologies should already be researched. If I had clicked 'yes' at
this point, I wouldn't be writing this.

Instead of scrapping the world and selecting the right menu options from a fresh
start, I can fix things as is.

Hitting the backtick key (`````) will toggle open the console. From there I can
run `/cheat`. And that's it. That will instantly mark all technologies as
researched. It also enables free instant crafting of any item without needing
the raw or intermediate resources.

Another option if I just want to unlock all technologies without fully putting
the game in _cheat_ mode is to execute the following lua command in the Factorio
console.

```lua
/c game.player.force.research_all_technologies()
```
