# Task Score
Task Score provides the score for an activity to the Opsive Behavior Designer (BD) Utility Selector. You configure the attributes and each attributes weight for each activity score. The BD Utility Selector then runs the activity with the highest score.   For example, a Retrieve HealthPack task could be configured with a high weighting for  health and a HealthPack being nearby.  As the health rating gets worse, the Retrieve HealthPack score gets higher  and eventually becomes the highest rated task for the Utility Selector.  Using weighted attribute task scores can provide more natural and intelligent behavior for an agent's decisions rather than having a tree of binary decisions. 

# Overview of Components

These components work with the Opsive Behavior Utility Selector:

- *Task Score* - This task provides the score for an activity to the BD Utility Selector based on the weights you assign to various attributes. The Utility Selector then runs the activity with the highest score.  Behavior Designer  global variables are used as the components of the score for a task.
- *Distance* - This component determines if key object types are visible (healthpacks, etc), calculates their distance, and makes their location and distance available as Behavior Designer variables. Rather than having a visibility cut-off based on in or out of field of view, this determines visibility based on a combination of angle and distance - the further to the side the object is, the lower the distance it is visible, while an object directly in front of the agent is visible further away.  This also provides an Explore attribute which indicates that no useful objects have been found.
- *Shooter Variables* - This component makes it easy to access variables used in an FPS type game in Behavior Designer such as Ammunition amount, Weapon strength, Health, Explore, and Anger level.  This component would be modified to track the scoring attributes specific to your particular game.
- *Anger* - This component increases the anger attribute of the Agent in the BD Attribute Manager when the agent is attacked. The Attribute Manager can be configured to decrease anger over time.  The initial value for anger in the Attribute Manager can range from zero for a passive agent to 100 for an aggresive agent.  

# 1. Task Score Component

Task Score is a Behavior Designer task which returns the score for a particular task group.  The Utility Selector will then run the task with the highest score.  

### Parameters
Any float Behavior designer global variable can be a component of the task score.   You can set a weighting from 0 to 1.0f for each attribute.  The  score is the sum of each attribute’s weight times the attribute’s value.    To provide a consistent basis for scoring:  
1. All attributes should be scaled from 0 to 100.0f.  
1. The sum of the weights should be equal to 1.0f for a high priority task, 0.9f for medium priority, and 0.8f for a low priority activity.  

### Reverse Scale
Since the Utility Selector chooses the task with the highest values, you sometimes need to reverse the scale for  a parameter to have it increase utility.  For example, as a player’s health gets lower, you might want to give a higher result for getting a healthpack.  You can do this by assigning a negative weight.  This is handled as a special case that reverses the scale (100-value) and applies the absolute value of the weight. 

### Setup
1. In the Behavior Designer edit window, add a Task Score Task for each activity under Utility Selector and connect it to the Utility Selector with the actions connected below it.  
1. In the BD Variables tab, add the Global Variables you want for your project (potentially the items in Sample Parameters above).  
1. In the BD Inspection window for each Task Score task, add the Global Variables you want for this task and add the weight for each variable. 
1. Setup Behavior Variables as described in that section.
1. If you want to use Anger in your score, setup Anger as described in that section.

### FPS Example

In this example for an FPS game, we are equal distance from the player and from the healthpack and our health is low. The attribute values are:  
  
*Health=20, HealthPackDistance=30, PlayerDistance=30*  

We have two tasks to score:  1) Seek Healthpack, and 2) Attack Player.  

*Task Score for “Seek Heathpack”*  
Weights are set to:  Health= -0.6, HealthPackDistance=-0.4  
**Score= 76** = (80 * 0.6) + (70 * 0.4)   (Note the Health score is reversed to 80 because the weight is negative.  The lower our health, the higher we want the score.  Distance is also reversed to reward being closer, not distant.)  
  
*Task Score for “Attack Player”*  
Weights are set to:  Health= 0.6, PlayerDistance=-0.4  
**Score= 40** = (20 * 0.6) + (70 * 0.4)   

In this case, the score for Seek Healthpack would be higher than Attack Player.  If Health was 80, the score for Attack Player would be higher.  

# 2. Distance Component  
The distance component tracks the distance from the agent to all objects with the tags listed below.  
*healthpack, player, ammo, weapon, ambush*  
It updates BD Global Variables with the same name (with Distance appended).
It determines if each object is visible by the agent and if visible, that object is marked as "found". If an object is not been seen for N seconds and is not static, the "found" flag is cleared.  The component calculates the distance to all currently "found" objects.  An object tag can have multiple objects, and the value will be for the current closest object with that tag.   0=close, 99=far, 100=not known.  

### Field of Vision  
Rather than having a fixed cutoff for whether an item is in or out of field of vision, objects toward the center can be seen further off, and off to the side must be closer to be considered visible.  

### Explore attribute  
This also provides an *Explore attribute* which is percent of target types not yet found.  The higher this value, the more useful exploring is.    
If a single object is found for the tag, the tag is considered found.  These are the tags tracked:   
*healthpack, player, ammo, weapon*

### Variables  
*HealthPackDistance* - 100=not found, 99=far, 0=near  
*playerDistance* - 100=not found, 99=far, 0=near  
*ammoDistance* - 100=not found, 99=far, 0=near  
*weaponDistance* - 100=not found, 99=far, 0=near  
*ambushDistance* - 100=not found, 99=far, 0=near  
*explore* - 100=100% of target types unfound, 0=0% of target types unfound.  

### Setup
*todo*   

# 3. Shooter Variables Component  
This Unity component makes it easy to access FPS type variables in Behavior Designer such as:  
*Ammo, Weapon, Health, Anger  

All variables are provided as floats scaled from 0 to 100.0f.   
The component would be modified to track other types of variables specific to a particular game.  

### Variables
*Target Health* - 100=healthy, 0=dead  
*Health* - 100=healthy, 0=dead  
*Anger* - 100=angry, 0=calm.  Damage to the agent increases anger.  Time decreases anger (based on the rate set in Attribute Manager)  
*Weapons* - 0 none, 100=most powerful weapon   
*Ammo* - 100=full, 0=empty  

### Setup  
1. Add the Behavior Variables component to your agent.

# 4. Anger Component

The anger component increases the anger attribute in Attribute Manager when the agent receives damage.

### Setup
1. Add the Anger component to your Agent.  Set the increment value for each damage attack in the Inspector.
1. Add an Anger attribute to your agent’s Attribute Manager with min/max of 0,100 and set auto decrement (if you choose).  Set the initial value to zero for a passive agent and to 100 for an aggresive agent or any value in between.  
