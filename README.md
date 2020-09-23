# Game Task Score
Using weighted attributes can simplify game design and provide more natural and intelligent behavior for an agent rather than having to create complicated trees of binary decisions. Task Score provides the score for an activity to the Behavior Designer (BD) Utility Selector based on the weights you assign to various attributes. The Utility Selector then runs the activity with the highest score.   For example, a task which retrieves a HealthPack would be configured with a high weighting for low health and a high weighting for a HealthPack being near.  As the health rating gets worse, the retrieve HealthPack gets a higher rating and eventually becomes the highest rated option for the Utility Selector.  

# Overview

This includes three components that work with the BD Utility Selector:

- *Task Score* - This BD task provides the score for an activity to the BD Utility Selector based on the weights you assign to various attributes. The Utility Selector then runs the activity with the highest score.  Any variable present in the BD Attribute Manager can be used as a component of the score for a task.
- *Anger* - This Unity component updates the anger attribute of the Agent in the BD Attribute Manager.  Anger can be one of the weighted attributes for an Attack Player task.  Anger is increased when the agent is attacked.  The Attribute Manager can be configured to decrease anger over time.  The initial value for anger can be set to zero for a passive agent or to 100 for an aggresive agent.  
- *Distance* - This Unity component determines if an object is visible, calculates the distance, and updates the Behavior Designer Attribute Manager. This offers a few features compared to standard distance calculations. Rather than having a strict field of view cut-off, this determines visibility based on a combination of angle  and distance.  An object directly in front of the agent will be visible further away.  The further to the side the object is, the lower the distance it will be visible.

# 1. Task Score

Task Score is a Behavior Designer task which returns the score for a particular task group.  The Utility Selector will then run the task with the highest score.  

### Parameters
Any attribute in BD Attibute Manager can be a component of the task score.  You can set a weighting from 0 to 1.0f for each attribute.  The  score is the sum of each attribute’s weight times the attribute’s value.  All attribute are scaled from 0 to 100.0f.  To provide a consistent basis for scoring between activities, the sum of the weights should be equal to 1.0f for a high priority task, 0.9f for medium priority, and 0.8f for a low priority activity.

### Reversed Scale
The Utility Selector chooses the task with the highest values, so sometimes you need to reverse the scale for  a parameter to have it increase utility.  For example, as a player’s health gets lower, you might want to give a higher result for getting a healthpack.  You can do this by assigning a negative weight.  This is handled as a special case that reverses the scale (100-value) and applies the absolute value of the weight. 

### Sample Parameters
Target Close:  100=close, 1=far, 0=not visible. Value is 100 - distance (capped at 99).  The target can be healthpack, weapon, ammo, player, etc.
Far Targets:  100=None found, 99=All far, 0=All near.  Average distance to all Targets.  The higher this value, the more useful, patrol/wander is.
Target Healthy: 100=healthy, 0=dead
AI Healthy: 100=healthy, 0=dead
AI Angry: 100=angry, 0=calm.  Damage to AI increases anger.  Time decreases anger (based on the rate set in Attribute Manager)
Weapons: 0 none, 10 club, 20 knife, 30 sword, 40 pistol, 50 shotgun, 60 automatic 
Ammo:  100=full, 0=empty

### Setup
In the Behior Designer edit window, add the Task Score task for each activity under Utility Selector.  Connect it to the Utility Selector.  In the BD Inspection window, add the Attributes you want for this task and add the weight for each attribute.  The attributes must use the same name as in the Attribute Manager (names are case sensitive).

### Example

In the example, we are equal distance from the player and from the healthpack and our health is low.  We have two tasks to score:  1) Seek Healthpack, and 2) Attack Player.  The attribute values are:
Health=20, Target Close(HealthPack)=70, Target Close(Player)=70

*Task Score for “Seek Heathpack” task*
Weight Set to:  Health= -0.6, Target Close(HealthPack)=0.4
Score= (80 * 0.6) + (70 * 0.4) = 76  (Note the Health score is reversed to 80 because the weight is negative.  The lower our health, the higher we want the score.)

*Task Score for “Attack Player” task*
Weight set to:  Health= 0.5, Target Close(Player)=0.4
Score= (20 * 0.5) + (70 * 0.4) = 38  (Note that the sum of weights is 0.9, so if everything is equal (equal distance to targets and health=50), the Seek Healthpack will be selected since it has total weights of 1.0.)

# 2. Anger

Anger is a component that reduces the anger attribute when the agent receives damage.

### Setup
Anger - Add the Anger component to your Agent.  Add an Anger attribute to your AI’s Attribute Manager with min/max of 0,100 and set auto decrement (if you choose).  Anger will be incremented when the AI takes damage.

# 3. Distance


