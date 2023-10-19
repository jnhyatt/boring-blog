+++
title = "Dev Log #1"
[taxonomies]
tags = [ "chain reaction" ]
+++

# Releasing is Hard

A few years ago, I released an Android game onto the Play store called Chain Reaction. It was heavily based on a game by Buddy-Matt Entertainment which has since been abandoned. It's a simple game, each player takes a turn placing an orb into either an empty cell or a cell they already have claimed. Each cell has a "critical mass" equal to the number of neighbors it has. When you fill a cell with enough orbs to reach critical mass, it explodes and the cells each spread to a neighbor. When those orbs hit, they both claim the neighbor cell for the player and add one orb to the cell. Then if those cells have hit critical mass, they also explode -- hence the name, Chain Reaction.

I was in high school and I really liked the game, but the UI was rather poor and it was somewhat lacking in features, so I reached out to the developer to ask if I could create a clone and extend it. He was all for it, so I made my own version but with *hexagons*. At first, I copied his pretty closely, but then I realized I could do fun stuff. I revamped the UI and gave users entirely too many options, then I shipped it. I eventually decided to monetize it, but only a couple years later.

Something I often reflect back on was the project timeline. I had the game playable in two days. The next day I had the game board looking nice. Within a week, I had a basic UI, an undo feature, and haptic/audio feedback.

I released it six months later.

I know going from prototype to releasable product takes time, but six months? What on earth needed to be done?

I have a couple theories. For starters, making the UI was a bit of an involved process because I had to draw nine-patches for sliders, text edits, buttons, icons, etc. I definitely overcomplicated the UI, but in my defense, I was 16. The original allowed you to customize player color and explosion sound on a per-player basis, and without a second thought I did the same. The original UI isn't what you'd call ergonomic -- the settings page has a list of players from 1 to 8, each of which brings you to a player settings page. This has the entries "Color Red Component", "Color Green Component", "Color Blue Component", "Sound", and I think a couple others. Clicking any of them would open up a dialog to edit the stored value. The end result was that if you wanted to change your color, you had to figure out its RGB value and then enter those values in one by one with no preview. The end result was that changing player colors was a bit of a doozy.

Such would not do for my work of art. So I sat down with Buddy-Matt's turd (apologies to him, I really loved that game) and began to polish. I added a color preview and put the RGB values on sliders that sat below the preview, with text edits to the sides of the sliders for fine-tuning. On top of that, I created a color select list with eight default colors to pick from. Switching a player's color was as simple as cliking on "Player 1" and then clicking on the color you wanted. If none of the colors sparked your interest, you could create a new custom color using the sliders. The new color would be saved into the color list and you could even edit it after the fact!

Content with my work, I moved on to overcomplicating the next system and polishing the next turd.

I deleted most of that code a year and a half ago.

It's still in source control, so I can always go back and admire my overenthusiasm. I made plenty more rookie mistakes, but I might end up detailing those in a later post. The moral of the story is that scope creep, bad requirements, lack of planning, and this-shouldn't-be-too-hard-itis are probably what turned a one-month game into a six-month game. Of course, in calling it a one-month game I embarrass myself, because I'm working on re-releasing it right now, and it's taken close to a year so far. In this case, it's more a result of me putting little time into it and having an obsession with rewriting things. That's more or less the purpose of this blog, though, so you could say I'm making progress. More status updates to come.
