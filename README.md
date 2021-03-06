# SuperTuxKart AI UT Fall 2019 Deep Learning Final Project

This was a project done for UT Austin's Deep Learning Course in Fall 2019. We discuss the approach we took for this project in this write up but have ommited the code to preserve the integrity of the assignment.


#### Team Members: Dale Soney, Eric Chen, Neel Kamal, Will Cray 

![](gameplay.gif)

*Gameplay showing ball chasing, positioning, and reversal when we pass the puck. Red is prediction, Green is truth*

## Introduction

For this final class project, we attempted to build an AI that would win a 2v2 hockey game against other AI players designed by classmates. Gameplay was done in a modified racing simulator built on SuperTuxKart. The hockey game we simulate is very similar to the popular game, Rocket League, which several of us have played. Taking from our real world experience, we attempted to bring a simplistic approach to having our AI score. Our goal was to keep the puck close to the opponent's goal as much as possible, in order to increase the amount of chances to score. From our experience in playing on a team in Rocket league, we also wanted to keep a certain distance between our 2 karts. This would allow for one kart to play defense or pick up the puck the team kart might lose. In this writeup, we will go over the different approaches we explored for this assignment and our development process.

## Exploration

Our team split into two sects: one working on a vision and controller approach and the other working on imitation AI. The imitation solution leveraged information from research in its application in Mario Kart [1] and on Super Smash Bros [2]. The MarioKart based approach leveraged DAgger to correct for deviation from the oracle. This aligned with what was taught in the lectures and seemed like an excellent solution for common problems associated with imitation learning. However, the simplicity of the approach in Super Smash Bros was appealing. The premise of their research was to successfully apply imitation learning to play video games in the simplest possible way. We modeled our imitation approach in a similar fashion: we would build an image classifier that received the past 3-5 image frames as input and output the optimal action for the next frame. Part of the action space is continuous ([-1,…, 1] and [0,…, 1]). In order to account for this, we would sample the continuous space to make it discrete. In the end, we had around 75 possible “classes” that our classifier could choose from, each corresponding to a different combination of actions our kart could take.

A critical component of the imitation learning approach was having enough quality data from an oracle for our model to imitate. Ideally, we could leverage the game’s AI (set to difficulty level 2, the strongest) to collect a massive amount of training data. We did not believe manually playing the game could cover every state the kart could find itself in a reasonable amount of time. In order to accomplish this, we modified some code from the Pystk code base [3] and were able to collect labeled data with the AI playing. However, for an unresolved reason, the AI only behaved as a quality oracle when we ran the game 1v1. When the game was executed with 2v2, the AI seemed to break and behave erratically. Around this time, the vision and controller-based approach became performant: we were scoring one goal against the AI 1v1 about 50% of the time and beating the AI about 10% of the time. We felt the imitation approach was promising, but we had no way of estimating how much time investment it would take for it to become as performant as the vision and controller technique. In the end, we decided to focus our efforts on the vision and controller-based approach for our final submission.

## Approach

Our vision based approach started with a similar FCN model to the one that was submitted for HW6. We used a setup with 2 (conv2d, batchnorm, relu) blocks per layer with 4 layers used. In order to generate labels for our training data, the pystk state instance was fed to each player kart instance, allowing them access to the true puck location. Using functions provided in previous assignments, we were able to both program the kart to follow the puck, and output the (x,y) screen coordinates the puck was mapped to. We repeated this process for several games, generating data from both team karts perspective and from both starting sides of the field. For points outside of our image, we experimented with leaving the values as were, clipping them to the nearest valid x value, and setting them to 0. In total, we collected around 10,000 labeled images for use in training our model.

Our initial model was able to predict the puck’s location on the screen with very high accuracy. It was able to track the puck more than half the court away, but took an incredibly long time to predict a puck location. Running the game with 2 agents demanding an output slowed the game down even further to 0.8 seconds per frame, which was well beyond what our target time of 0.1s/frame. We did additional testing with a simplified version of the model with half as many layers, which brought down the runtime to 0.4 seconds per frame. Finally, we turned towards the previous HW’s solution for inspiration, which was a much more simple FCN. Stripping away more than half the layers and parameters, accuracy did worsen, but model outputs were much faster (around the desired 0.1s per frame).

In order to see how our controller would work under ideal conditions, we built a separate testing case that used the true location of the puck, instead of our model prediction. This way, we were able to test several controller ideas while we were training and improving our models. We later substituted out the true location of the puck with our model output and verified if our strategies worked. Key ideas and features we built include:
Rescue/Recovery - We initiated a state for recovery if the kart was detected to be stuck along the wall or inside one of the goals for a certain amount of frames. Recovery involved either backing up, or reversing towards our own goal. Without this, the AI was able to chase down the ball and score without any resistance.
Shot alignment - We attempted to position our karts at a better angle to score. When the puck was in front of our kart, we projected where our kart would be if it were to keep on going straight. Depending on the scenario, slight course adjustments would  be made in order to get the puck in between the kart and goal.
Avoiding long turns - When the puck is detected moving below a certain y value on the screen, we established that we were not pushing the puck, but rather going past it. This feature saved time spent turning around and chasing the ball by simply reversing and repositioning.

## Conclusion

Our basic strategy of following the puck and adjusting steering based on court position performed remarkably well vs AI and our classmates. Keeping the puck constantly moving made it more difficult for our opponents to score, and we would rarely lose control of the ball. We submitted this method to the last tournament, which brought us to 2nd place.

While we have made several improvements to our controller and model, we continue to struggle with scoring consistently against the in game AI. Here, we felt the limits of a vision+controller based approach, as even though we are able to accurately detect where the puck is on the screen, hand coding a strategy to account for countless situations proved incredibly difficult. Scoring techniques that worked in theory struggled under actual game situations, with  karts, items, and other factors interfering with the game plan.

If we had more time, we felt that utilizing imitation learning from built in game AI on top of our puck recognition model could have yielded more promising results. Moving forward, we hope to continue to learn more about how modern self driving techniques integrate both vision and imitation learning into their models.

References

[1] http://cs231n.stanford.edu/reports/2017/pdfs/624.pdf

[2] https://arxiv.org/pdf/1702.05663.pdf

[3] https://github.com/philkr/pystk/blob/master/examples/play.py
