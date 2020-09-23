# Game Task Score
These components are used to create task utility scores for game development in Unity and Opsive Game Behavior Designer.  The Utility Selector uses these scores to select which task to run.  Each task has a score based on the key attributes for that task with each weighted.  Using weighted attributes can simplify game design and provide more natural and intelligent behavior for an agent.

# Overview

This includes three components that work with the Opsive Behavior Designer Utility Selector:

- Task Score - This Behavior Designer task provides the score for an activity to the Behavior Designer Utility Selector based on the weights you assign to various attributes. The Utility Selector then runs the activity with the highest score.
- Anger - This Unity component updates the anger attribute of the object.  Anger is increased when the object is attacked and decreases over time.
- Distance - This Unity component determines if an object is visible, calculates the distance, and updates the Behavior Designer Attribute Manager.

# 1. Task Score

Task Score is a Behavior Designer task which returns the score for a particular task group.  The Utility Selector will then run the task with the highest score.  The Task Score is placed on each task group and is configured with the key attributes for that task group and their weightings.  For example a task which retrieves a HealthPack would be configured with a high weighting for low health and a high weighting for a HealthPack being near.

### Parameters
You can set a weighting from 0 to 1.0f for each parameter.  The returned score is the sum of each parameter’s weight times the parameter’s value.  All parameters are scaled from 0 to 100.0f.  To provide a consistent basis for scoring between activities, the sum of the weights should be equal to 1.0f for a high priority task, 0.9f for medium priority, and 0.8f for a low priority activity.

### Sample Parameters
Target Close:  100=close, 1=far, 0=not visible. Value is 100 - distance (capped at 99).  The target can be healthpack, weapon, ammo, player, etc.
Far Targets:  100=None found, 99=All far, 0=All near.  Average distance to all Targets.  The higher this value, the more useful, patrol/wander is.
Target Healthy: 100=healthy, 0=dead
AI Healthy: 100=healthy, 0=dead
AI Angry: 100=angry, 0=calm.  Damage to AI increases anger.  Time decreases anger (based on the rate set in Attribute Manager)
Weapons: 0 none, 10 club, 20 knife, 30 sword, 40 pistol, 50 shotgun, 60 automatic 
Ammo:  100=full, 0=empty

### Reversed Scale
The Utility Selector chooses the task with the highest values, so sometimes you need to reverse the scale for  a parameter to have it increase utility.  For example, as a player’s health gets higher, you can easily use this to give a higher result for “Attack”.  However, as a player’s health gets lower, you might want to give a higher result for getting a healthpack.  You can do this by assigning a negative weight.  This is handled as a special case that reverses the scale of the parameter. The parameter is treated as (100-value) and the absolute value of the weight is applied. 

### Returns
The weighted value is returned to the Utility Selector with the GetUtility method
DemoAgent.Health, DemoAgent.Ammo, 

### Setup
Anger - If you are going to use Anger, you need to add an Anger attribute to your AI’s Attribute Manager with min/max of 0,100 and set auto decrement (if you choose) and add the Anger component to your AI.  Anger will be incremented when the AI takes damage.

### Example

In the example, we are equal distance from the player and from the healthpack and our health is low.  We have two tasks to score:  1) Seek Healthpack, and 2) Attack Player.  The parameter values are:
Health=20, Target Close(HealthPack)=70, Target Close(Player)=70

Evaluator for “Seek Heathpack” task
Weight Set to:  Health= -0.6, Target Close(HealthPack)=0.4
Score= (80 * 0.6) + (70 * 0.4) = 76  (Note the Health score is reversed to 80 because the weight is negative.  The lower our health, the higher we want the score.)

Evaluator for “Attack Player” task
Weight set to:  Health= 0.5, Target Close(Player)=0.4
Score= (20 * 0.5) + (70 * 0.4) = 38  (Note that the sum of weights is 0.9, so if everything is equal (equal distance to targets and health=50), the Seek Healthpack will be selected since it has total weights of 1.0.)

# 2. Anger

