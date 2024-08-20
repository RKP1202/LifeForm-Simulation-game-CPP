
# C++ PROJECT #

## LifeForm Simulation game ##

This project is to complete the simulation infrastructure that we get from the instructor and also to invent at least one new LifeForm and simulate the LifeForm's existance (or evolution) of that LifeForm.

### Overview: ###
  1. Events: We use the make_procedure function to create function objects as described in class. Events can then be dynamically allocated. The Event constructor takes a time and a procedure. The procedure will be called that many time units into the future (the time is relative, not absolute). Creating an event automatically puts the event in the central event queue. Events automatically delete themselves when they occur. If we delete an event before it occurs it automatically removes itself from the event queue.
  2. Region of space(quadtree): We use the QuadTree\<LifeForm*\> data structure to represent space. 
  3. How to move around:
- LifeForm::update_position(void) - computes the new position, charge for energy consumed.
- LifeForm::border_cross(void) - movement event handler function. It calls update_position and then schedules the next movement event.
- LifeForm::region_resize(void) - this function will be a callback from the QuadTree. When another object is created nearby, the object needs to determine the next possible time that they could collide. The QuadTree knows when objects are inserted inside it, so it can invoke this callback function on your objects. We have to have region_resize cancel any pending border crossing events, update the position, and schedule the next movement event (since the region has changed size, we will encounter the boundary at a different time than before.)
- LifeForm::set_course and LifeForm::set_speed - These functions should cancel any pending border_cross event, update the current position of the object and then schedule a new border_cross event based on the new course and speed. Note that an object that is stationary will never cross a border.
  4. Creation of LifeForms: LifeForms cannot be created spontaneously and just expect to be simulated. There are two ways that LifeForms can be created. First, they can be created by the LifeForm::create_life method during initialization. The other way that objects can be created is by calling reproduce.

### Rules of the Game ###
  1. Encounter: If two creatures ever get within one unit of distance of each other, they have an enounter. The encounter method returns either EAT or IGNORE. Since there are two LifeForms colliding, there are four cases; IGNORE/IGNORE, EAT/IGNORE, IGNORE/EAT and EAT/EAT. Provided at least one LifeForm attempts to eat, we need to generate a random number and compare the result to eat_success_chance(eater-\>energy, eatee-\>energy).
  2. Names: There are two functions, player_name and species_name. By default, player_name returns the species name. The player name is used during the contest to select the winning LifeForms among all students in the class. The player_name is used only by the contest and only to print out which LifeForms are surviving (and which have become extinct). The species_name is used when objects collide with or perceive each other. The species_name can be anything we'd like. (We can lie to enemies in the species_name!!)
  3. Perception: The perceive routine can be invoked by any derived LifeForm class at any time. max_perceive_range and min_perceive_range define the maximum and minimum radii for the perception circle. Each time an object attempts to perceive, it should be charged the perceive_cost. Note that perceive_cost is a function of the radius. The perceive function should return a list of ObjInfos. Each ObjInfo includes the name, bearing, speed, health and distance to the object. One ObjInfo should be placed into the list for every object within the prescribed radius.
  4. Eating: Objects eat each other according to their desire (indicated by the return value of encounter) and their eat_success_chance (defined in Params.h). If object A wants to eat object B, then the simulator should generate a random number and compare it to the eat_success_chance(A, B). If the random number is less than the eat success chance, then A gets to eat B. B could also be trying to eat A at the same time. Params.h includes a variable called encounter_strategy which explains how to break the tie. Once we've figured out who got to eat whom, we have to award the victor the spoils. The energy is awarded to the eatee after exactly digestion_time time units have passed (create an event). The energy awarded is reduced by the eat_efficiency multiplier. (Since eat_efficiency is less than 1, it is usually not a good idea to eat your own young!!)
  5. Aging: Objects must eventually die if they don't eat. This is true even if they act like rocks and don't move and don't perceive. Not all LifeForms are born at the same time (some are children of other LifeForms) and so they shouldn't be penalized at the same time.
  6. Reproduction: When a LifeForm reproduces, the energy from the parent LifeForm is divided in half. This amount of energy is given to both the parent and the child. Then, the fractional reproduce_cost is subtracted from both the parent and the child. For example, a LifeForm with 100 energy that reproduces will produce a child that has 50 * (1.0 - reproduce_cost) energy. The parent will have the same energy as the child. As a special rule, do not allow an object to reproduce faster than min_reproduce_time.
  
Here is how I implemented [LifeForm.cpp](./lab3/LifeForm.cpp).
