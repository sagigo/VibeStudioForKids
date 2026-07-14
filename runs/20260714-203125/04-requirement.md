# Requirement

## Original request
There is a car that has a number on it. On the road there are exercises and I need to collect only the exercises that come out like the number on the car. Exercises like multiplication, division, addition and subtraction.

Game flow: Need to turn right and left to take the correct exercises whose result is on the car.

How to play with the keyboard: Right arrow to turn right, left arrow to turn left.

## Summary
A browser-based arcade math game: a space race car flies down a glowing multi-lane space highway with a target number displayed on it. Math exercise tiles float toward the player in the lanes. The player steers left/right between lanes to collect only the tiles whose answer matches the number on their car, while avoiding wrong-answer tiles. The player has 3 lives; hitting a wrong-answer tile costs a life, and the game ends when all lives are lost. The goal is to survive as long as possible and rack up the highest score, with the game getting faster and harder as the score climbs.

## Clarified details
- **Involvement**: Kid chose to help decide the details together (not "surprise me").
- **Mistake consequence**: Driving into a wrong exercise tile costs one life (not an instant game over).
- **Lives**: 3 lives total. Game ends (game over) when lives reach zero.
- **Math scope**: Addition and subtraction only, with numbers up to 100 (both the target number on the car and the exercises on the road stay within this range). No multiplication or division, despite being mentioned in the original idea - simplified per the kid's choice.
- **Game mode**: Survive & score mode. There is no separate "win" condition or target score - the player plays until they lose all 3 lives.
- **Scoring**: Score = number of correct exercise tiles collected.
- **Difficulty progression**: As the score increases, the game speeds up (tiles/road move faster), making the game progressively harder over time.
- **Visual theme**: Space race setting - a glowing space highway with a starfield background, planets, and neon-colored lanes.
- **Controls**: Left arrow key moves the car one lane to the left; right arrow key moves the car one lane to the right, to steer between lanes and collect the correct exercise tile.

## Acceptance criteria
- The game runs in a web browser as a static, self-contained game (HTML/CSS/JS), no server or login required.
- A car is shown on a multi-lane space highway, displaying a target number (within 0-100) that changes periodically (e.g., each time it's matched, or on a timer).
- Math exercise tiles (addition and subtraction problems using numbers up to 100, e.g. "57 + 12", "88 - 30") continuously appear in the lanes ahead and scroll/move toward the car.
- At least one visible tile's answer matches the current target number on the car at any given time, so the level is always solvable.
- Pressing the left arrow key moves the car one lane left; pressing the right arrow key moves the car one lane right, constrained to the available lanes (car cannot move off the road).
- Colliding with a tile whose answer matches the car's target number: counts as a correct collection, increases the score by 1, and a new target number/tile set appears.
- Colliding with a tile whose answer does NOT match the target number: costs the player one life (out of 3 total) and the tile is removed/passed.
- The current score and remaining lives (e.g., as icons or a counter) are visible on screen at all times during play.
- As the score increases, the speed at which tiles/road move increases, making the game progressively more challenging.
- When lives reach zero, the game ends and shows a "Game Over" screen displaying the final score, with an option to restart/play again.
- The visual style is a space theme: a starfield/space background, glowing/neon-colored lane dividers or tiles, and a sense of racing forward through space.
- The game is playable entirely via keyboard (left/right arrow keys); no mouse or touch input is required for core play.
