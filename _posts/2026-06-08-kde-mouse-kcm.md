---
title: "Contributing to KDE Plasma - KCM Mouse"
date: 2026-06-09 22:33:14 -0300
categories: [KDE, Patches]
tags: [KDE, linux, onboarding, patch, contribution]
---

## Background
This posts continues the MAC0470 - Free Software Development subject journey, relating specifically to its second stage.

This time around, we were supposed to choose an open source project we'd like to contribute to. If you've been paying attention to these posts, you'll know by now that everything about to be related was done in tandem with André Jun Hirata and Guilherme Santos Gabriel.
They are the biggest reason why we wound up choosing KDE as they use it daily and we as a whole were very interested in the project. When our teacher, PhD Paulo Roberto Miranda Meirelles, put us in contact with Farid Abdelnour, an active member of the KDE community, the decision was a no-brainer.

The three of us and Farid gathered for a first "onboarding" meeting in which he introduced us to the whole KDE universe and already started showing some of the ongoing projects we could join.

## The task we chose
After hearing all Farid had to say, we opted to partake in the KCM Mouse project, whose basic premise was to introduce to KDE Plasma functionalities similar to the ones offered by libratbag or Piper, i.e. general mapping and configuration/customization for mice.
We took particular interes in this project as we were told a succesful contribution could mean its integration into the all fresh Steam Deck :)

Heres how KCM's current configuration page looks like:

![KCM do mouse atual 1](/assets/img/posts/kde-mouse-kcm/mouse-kcm-current-1.png)

*Current mouse customization page*

And this is what Piper, implemented with libratbag, looks like:

![Piper resolution](/assets/img/posts/kde-mouse-kcm/piper-resolutionpage.png)
![Piper buttons](/assets/img/posts/kde-mouse-kcm/piper-buttonpage.png)
![Piper led](/assets/img/posts/kde-mouse-kcm/piper-ledpage.png)

*Piper — libratbag's frontend*
Remember, this is a good clue as to what we wish for the final product.
Mind you, though, we were highly suggested not to use libratbag, but rather develop a native Plasma backend.

Initially, what we did was explore the KCM mice repo, check libratbag's code and tag along Kwin's Matrix's discussions.

## Matrix Discussion
Speaking of which, let's dive into what was our biggest interaction with the KDE community thus far.

After analyzing Kwin's backend, we realized some of the mouse model information getters were exposed by the API. Naturally, we wondered if there were any plans in place for a KDE-hosted database.
After inquiring Farid, he instructed us to post our question to the Matrix discussion directly.

Jakob, a KDE developer, promptly answered. Basically, there isn't a definitive answer and there is space for design.

He also introduced a few tension points that should be taken into account:

- The aforementioned libratbag dependency;
- The design choice between software-based and firmware-based button-mapping;
- DPI vs Pointer Speed as two different ways to achieve the same result.

## Refining the approach
Having all this information, we turned to Farid once again.

His feedback: define an issue proposal and post it to KDE's Invent so that a developer can approve it and Farid can forward it to the design team.

And here it is: [Proposta no Invent](https://invent.kde.org/teams/goals/we-care-about-your-input/-/work_items/18)

Regarding the architectural choices, here are the decisions we made:

- Software-based button remapping through Kwin's ButtonRebinds
- Utilization of a KDE-hosted device database
- Device grouping to avoid gaming mice from registering as multiple devices
- Optional libratbag integration

## Roadmap
The proposed steps towards a functional and improved interface:

1. Extend the KWin D-Bus API to expose the libinput group ID
2. Group devices by group ID in the KCM list
3. Set up the database by mirroring libratbag/Piper
4. Integrate mouse icons using vendor/product IDs
5. Improve the remapping UI: mouse→mouse, messages for unsupported mappings
6. Add optional integration with libratbag for LED settings
7. Add DPI support via libratbag; UI approach to be determined

## Next steps
The proposal is posted in the link above and is waiting for feedback. So far, we've got a hint that we might be on the rigth track:

![Feedback inicial](/assets/img/posts/kde-mouse-kcm/brief-feedback.jpg)

*Jakob's take on our roadmap*
