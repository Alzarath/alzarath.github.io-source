---
title: Movement
date: 2019-03-09 10:09:28
---
I'm in my second (or third, depending on how you look at it) Game Design class since I started college. In this class I'm working with a group of two others to create a game in Unity that's been designed and conceptualized from the previous semester. The past couple weeks we've been working on getting the core of the game working. This includes the following:

* Player movement
* Item Collection
* Collision

The item collection and collision parts aren't that interesting, but I'd like to talk about my discoveries in player movement. Player movement is accomplished by *moving* the player in the direction of a vector. We had the player moving along the X and Z axis when the player presses the designated movement buttons. Unfortunately this means the players rotation does not affect the movement direction.

We had the player rotating along with the mouse in a pretty simple way. The question was how we got the player to move in the direction they are facing. There didn't seem to be a convenient Vector3 method to rotate a vector along any one axis depending on the rotation of the player, so what were we to do?

I decided to change the x and z vector manually depending on the rotation of the player. It may seem obvious now, but in reality it took hours of trial-and-error. Because I want to learn from the results, I've decided to describe not only to you why it works the way it does, but to myself.

Early on, I decided to try to get the player's `transform.rotation`. This was actually working fairly well, until I realized that it seems to use a sine wave, which meant the direction is not accurate for horizontal movement. The value of 0.5 was at about 60° rather than 90°. This obviously caused issues. My first thought was to figure out how to somehow counter-act the sine, but I thought of a better, easier idea, which was just use the transform's `eulerAngle`.

{% asset_img OriginalAngle.png %}

The `eulerAngle` uses degrees from approximately 0 to 360. The first question was how we translate from these degrees to numbers that would make the player move in the desired direction. I had a revelation that I could invert a capped value by subtracting the value from the cap. This means I could take the value and, when it passes 180, I could make it start going back toward 0 by subtracting the value from 360 (`value = 360 - value`)

{% asset_img 180Angle.png %}

Now I have a range of values that go from 0 to 180, then back to 0 when doing a complete turn. This is closer to what I want, but not quite there. I applied the same logic, subtracting the value from 180 when it exceeded 90° (`value = 180 - value`). This means it goes 0-90-0-90-0. This gives me meaningful numbers that I can use to adjust the speed of the z value so the player will move faster when moving "forward" and slower when moving "sideways".

{% asset_img 90Angle.png %}

I divided the value by 90 (`value = value / 90`) so it represents "intensity" of movement from 0 to 1 which I'll then multiply by the speed of the movement. This works great for horizontal movement, since the intensity is focused on the x axis, but not so well for vertical movement. It also has a major issue with only being able to travel in one direction.

{% asset_img Horiz1.png %}

I resolved the uni-directional nature by multiplying by -1 in the initial `value > 180` test. The only problem this gives is subtracting -90 from 180 certainly does not give the desired value, so I had to multiply the 180 by the current value's "sign" (`value = 180 * Mathf.Sign(value) - value`).

{% asset_img Horiz2.png %}

What about vertical movement, though? It was much easier than I'd have expected. Take the initial value after the 360° test, divide it by 90 which sets the values to the intensity, subtract it by 1 so that the values hit the direction, and multiply it by -1 so it's facing the direction (`value = (value / 90 - 1) * -1`).

{% asset_img 180Angle.png %} {% asset_img Vert1.png %} {% asset_img Vert2.png %} {% asset_img Vert3.png %}

While the respective key is being pressed, I add them to the movement vector. I do so separately for the horizontal and vertical movement so the player can move diagonally, then normalize the values so the player isn't going super fast while moving diagonally.

I set the `vector.x` and `vector.z` of the total movement vector to this calculated vector, supply the CharacterController's Move method with it, and the movement works without issue. Allowing the player to reverse or go left is as simple as multiplying the value by -1 while the player is holding the backward or left key. And with that, the movement is complete.
