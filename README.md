# Task Score
**Task Score** provides a score for a task in a Unity game to the Opsive Behavior Designer (BD) **Utility Selector** based on attributes that you configure. For each task, you configure which attributes to use and how much to weight each attribute. The Opsive BD Utility Selector will then run the task with the highest Task Score.   For example, a Retrieve HealthPack Score could be configured with a high weighting for poor health and small distance to a HealthPack.  As the health rating gets worse, the Retrieve HealthPack score gets higher and if there is a healthpack nearby becomes the highest rated task for the Utility Selector.  This approach of using weighted attribute task scores can be easier and provide more natural behavior for an agent's decisions versus using a decision tree for every case. *NOTE: These components require Opsive Behavior Designer but are not affiliated with Opsive or supported by Opsive.*  
:no_entry_sign: Indicates a feature has not yet been implemented  

### Table of Contents
**[Overview of Components](#Overview-of-Components)**<br>
**[Task Score Component](#Task-Score-Component)**<br>
**[Distance Script](#Distance-Script)**<br>
**[FPS Variables Script](#FPS-Variables-Script)**<br>
**[Troubleshooting](#Troubleshooting)**<br>

# Overview of Components

These three components work together to provide a task score to Opsive Behavior Designer **Utility Selector**:

- *Task Score* - This Behavior Designer Decorator provides the score for an activity to the BD Utility Selector based on the weights you assign to various attributes. The Utility Selector then runs the activity with the highest score.  Any Behavior Designer global variable can be used as the attributes of the score for a task.
- *Distance* - This script determines if objects with specific tags are visible (player, healthpack, etc), calculates their distance, and makes their location and distance available as Behavior Designer variables. Distances are normalized from 0 (near) to 100 (far). This also provides an *Explore* attribute which indicates whether key objects haven't been found yet and it would be useful to explore.  
- *FPS Variables* - This is a sample script that makes it easy for Behavior Designer to access variables used in a basic FPS type game, such as: Ammunition amount, Weapon strength, Health, Explore, and Anger level.  All variables are floats normalized from 0 to 100. This script would be modified or replaced to track the task scoring attributes specific to your particular game and place them in BD global variables. 

*Behavior Designer from Opsive is required for Task Score to work and Ultimate Character Controller from Opsive is required for FPS/AI movement.*

# *Component Details*  

# Task Score Component

Task Score is a Behavior Designer Decorator which returns the score for a particular activity based on the weights you assign to various attributes.  The Utility Selector will then run the task with the highest score.  Task Score is configured with different attributes and weights for each activity.  
*Figure 1 - Utility selector with Task Score*
![x utilitySelector](images/utilitySelector.png)  

### Task Scoring Example

In this example for an FPS game, the agent currently has low health and is 30 units from both the player and from the healthpack. The Behavior variables values currently are:  
  
*Health=20, HealthPackDistance=30, PlayerDistance=30*  

We have two tasks to score and decide which to run:  1) Seek Healthpack, and 2) Attack Player.  

*Task Score for “Seek Heathpack”*  
Task Score weights for this were configured to:  Health= -0.6, HealthPackDistance=-0.4  

(Note the Health score is reversed to 80 because the weight is negative (see Reverse Scale below).  The lower our health, the higher we want the score.  Distance is also reversed to reward being closer, not distant.   
**Score= 76** = (80 * 0.6) + (70 * 0.4)      
  
*Task Score for “Attack Player”*  
Task Score weights for this were configured to:   Health= 0.6, PlayerDistance=-0.4  
**Score= 40** = (20 * 0.6) + (70 * 0.4)   

In this case, the score for Seek Healthpack is higher than Attack Player.  If Health improved, the score for Attack Player would be higher.  This is a simplified example, in a real score you might also want to include ammo amount, weapon strength, anger, etc.

### Attributes
Any float Behavior Designer global variable can be an attribute of the task score.   You can set a weighting from 0 to 1.0f for each attribute.  The  score is the sum of each attribute’s weight times the attribute’s value.    To provide a consistent basis for scoring:  
1. All attributes should be scaled from 0 to 100.0f. The scripts in this package already do that for distance and FPS Variables. 
1. The sum of the weights of all the attributes should be equal to 1.0f for a high priority task, 0.9f for medium priority, and 0.8f for a low priority activity.  
*Figure 2 - Behavior Designer Inspector tab for Task Score for Seek Healthpack*  
The second attribute is distance to the healthpack.  Note that an upper bound of 80 is set.  If health is over 80, this will return a score of zero no matter how close we are to a healthpack.  

![x Healthscore](images/healthScore.png)  

### Reversed Scale
Since the Utility Selector chooses the task with the highest values, you sometimes need to reverse the scale of  an attribute to have it increase utility.  For example, as a player’s health gets *lower*, you might want to give a *higher* result for getting a healthpack.  You can do this by assigning a negative weight.  This is handled as a special case that reverses the scale (100-value) and applies the absolute value of the weight.  Use a negative weight on distance when you want the score to be higher when you are closer, use a negative weight on health when you want the score to be higher when you are weaker, etc. 

### Upper and Lower Bounds
You can also specify that an attribute MUST be *lower* than a upper bound or *greater* than a lower bound.  If that attribute does not fall in those bounds a task score of zero is returned *regardless of any other attributes*.  For example, you can configure that playerDistance must be lower than 2 for a Melee attack or else return a score of 0 regardless of the value of any other attribute.
*Note 1: The bound is checked *without* reversing the scale.*  
*Note 2: Upper bound and lower bounds of zero are ignored.*    

### Success Status of Child Task  
The Success/Failure of the child activity is monitored.  If the child activity fails, the score is reduced for the next N seonds.  For example, a "Flee Player" task is called but the agent cannot move further from player and the task returns fail, or a "Pickup Item" task is called but the Agent is unable to pick up the item.

### Setup  
1. Ensure you can run the demos in Opsvive's Behavior Designer https://opsive.com/support/documentation/behavior-designer/integrations/opsive-character-controllers/  
1. Before setting up this component follow the Setup instructions for the Distance component, and FPS Variables components below.
1. Copy Task Score.cs to your assets folder.
1. In the Behavior Designer edit window, add a **Task Score** Task for each activity under **Utility Selector** and connect it to the **Utility Selector**.  Add the appropriate action for the task below Task Score.  
1. In the BD Inspection window for each **Task Score** task, add the Global Variables you want for this task and add the weight for each variable. See figure 2 above. You can also set a "greater than" lower bound and a "less than" upper bound for each attribute.  If the bound is not met, zero is returned for the score.
1.  You must have the same number of entries for the weights, and upper and lower bounds.

# Distance Script  
The Distance script tracks the distance from the agent to all objects that have the tags you have added in the Inspector and tracks their current location.  
It updates BD Global Variables with the same name (with Distance appended and Location appended).  
*A tag MUST be applied to game objects you want to track*  
*For a pickup item, the tag MUST be applied to the item that has the pickup script.*  
This script determines if each object is visible by the agent and if visible, that object is considered "found". The "found" flag is cleared and distance is marked unknown if an object has not been seen for N seconds *and the object is not static*.  The script calculates the distance to all currently "found" objects.  A tag (e.g. Healthpack) can have multiple objects, and the distance will be for the closest object with that tag.   Distance is normalized from 0 to 100, 0=close, 100=far. *NOTE: 1000 is returned for objects that haven't been found.*  This will create very high scores for not found items, which will  make a task using that distance unlikely to run.  A task can also set an upper limit of 100 so it will not run for items that have not been found.  The Distance script also stores the location of each known object.  The location is NOT cleared when an item is no longer visible.  Only the Distance variable should be used to determine visibility.

### Tags
The distance component tracks the distance from the agent to objects that have the tags you have added in the Inspector.  For an FPS these
might be something like:  *healthpack, player, ammo, weapon, etc. 

### Field of Vision fall off
Rather than having a fixed angle cutoff with an item either in or out of field of vision, this script allows objects toward the center of view to be visible further off, and objects off to the side must be closer to be visible.  This script also allows marks an item behind you as visible if it is close (anything within minDistance is visible even if behind you).  If DebugFlag is set, this will log the cutoff distance for each angle of field of vision.
The calculation for degrees from 0-90 is the following (above 90 minDistance is used):  

*viewableDistance = Mathf.Max(maxDistance - (angle * angleWeight), minDistance)*  

maxDistance, angleWeight, and minDistance are configurable in the Inspector. Angle of 0 is straight ahead and 90 is directly to the side.  

*Figure 3 - Viewable distance with maximum set to 30 and minimum set to 3*
![x viewableDistance](images/viewableDistance2.png)  

### Explore attribute  
This also provides an *Explore attribute* which is based on how many key components haven't yet been found.  The higher this value, the more useful exploring is.    
If a single object is found for the tag, the tag is considered found.  These are the tags tracked for the explore value:   
*healthpack, player, ammo, weapon*

### Distance Variables  
Example for Player being added as a Tag to track, the following BD Global variables would be updated for the closest object with the Player tag.
*PlayerDistance* - 1000=not found, 100=far, 0=near   
*PlayerLocation* - There is also a location Vector3 Variable for each

*Explore* - 100=100% of target types unfound, 0=0% of target types unfound.  

### Setup
1. Copy Distance.cs to your assets folder and add it to your Agent.  
2. In the Inspector for Distance, add all the tags you want to track.
3. Assign the tags  above to the components you want to track.  
4. In the Behavior Designer Variables tab, add the Global Variables you want for your project for the tags you have added.  The name must be the tag with "Distance" appended and "Location" appended.  The type should be float for distance and Vector3 for location.
*Figure 4 - Behavior Designer Variables tab*
![x bdVariables](images/bdVariables.png)

# FPS Variables Script  
This  script makes it easy to access normalized FPS type variables in Behavior Designer such as:  
*Ammo, MeleeWeapons, RangeWeapons, Health, Anger*  

All variables are provided as floats scaled from 0 to 100.0f.   
This component should be modified to track the variables for scoring specific to your game. 

### Anger  
This also maintains the Anger score and increases the anger attribute in Attribute Manager whenever the agent receives damage.  The BD Attribute Manager can be configured to decrease anger over time.  The initial value for anger in the Attribute Manager can range from zero for a peaceful agent to 100 for an aggresive agent.  

### FPS Variables
*Health* - 100=healthy, 0=dead  Agent Health   
*Anger* - 100=angry, 0=calm.  Damage to the agent increases anger.  Time decreases anger (based on the rate set in Attribute Manager)   
*WeaponStrength* -  0 none, 100=most powerful weapon.  Power of equipped weapon. For example: 40 Club, 60 Knife, 65 Sword, 95 Rocket Launcher  
*MeleeWeapons* - :no_entry_sign: 0 none, 100=most powerful weapon.  Power of equipped weapon. For example: 40 Club, 60 Knife, 65 Sword.  
*RangeWeapons* - :no_entry_sign: 0 none, 100=most powerful weapon.  Power of equipped weapon. For example:  75 Pistol, 85 Assault Rifle 95 Rocket Launcher.      
*Ammo* - 100=full, 0=empty.  *Note: this is the percent of "adequate" ammunition, not the number of bullets.*  A rocket launcher with 10 rockets would return a score of 100 while an assault rifle with 10 bullets would return a score of 30.

### Setup  
1. Copy the FPSVariables.cs script to your assets and add it to your agent.
1. Set the Anger increment value for a damage attack in the Inspector.
1. In the BD Variables tab, add the Global Variables you need for your project from the FPS Variables list above.  
1. Add an Anger attribute to your agent’s Attribute Manager with min/max of 0,100 and set auto decrement (if you choose).  Set the initial value to zero for a passive agent and to 100 for an aggresive agent or any value in between.  

# Troubleshooting  
1. Behavior Designer installation - make sure you can successfully run the BD demos.
1. Debug info - In the Behavior Designer Inspector for the Task Score task, enable the magnifying glass on "Sc".  This will display the value of the task score on the Behavior Diagram.
1. Debug info - For each script, enable DebugFlag to log debugging information.
1. Tags - All the game objects you want to track must have a tag (weapon, player, healthpack, etc) for the distance calculator.  Make sure the player is tagged, healthpacks are tagged, weapons are tagged.  The weapon names must match a name listed in WeaponDict in FPSVariables.cs. For a pickup item, the tag MUST be applied to the item with the pickup script.** 
1. Reverse Scale - Use a Reverse Scale (negative weight) if you want a high score for a *close* distance, *weak* health etc.  **Most  items will use a reverse score (negative weights).** Double check any items with positive weight.
1. Bounds - Set upper/lower bounds for scores where appropriate.  For example, attack player should have a minimum ammo greater than zero.  With zero ammo it will generate a low score but that might still be the best score available.  This state will cause an attack, which will do nothing and will likely be chosen again.
1. Weights - The sum of the weights should be equal to 1.0f for a high priority task, 0.9f for medium priority, and 0.8f for a low priority activity.  
2. New Variables - If you add your own variable in FPSVariables for weighting make sure it is normalized from 0.0f to 100.0f.
3. Action - The action under a task, *MUST end up altering at least one of the scores for that task* otherwise you will get stuck running the same task.
4. Targets - Make sure that Seek and Flee have been set up to have correct targets.
5. Default Task - You should have one Task which will run when everything else has a low score. This task should not have any upper or lower bounds.  This might be an explore task.
6. NavMesh - Make sure you have an up to date bake of the AI Navmesh for your scene.
7. Utility Selector - The Task Score tasks MUST be connected to a **Utility Selector** task.  
8. **TODO** :no_entry_sign: Use the spreadsheet below to determine optimal weights.  The spreadsheet allows you to enter all the tasks and will calculate their score as you change variable values.

*Copyright 2021 Michael Herbert*
