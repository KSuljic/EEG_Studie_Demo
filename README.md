# EEG Study Demo


Author: Kenan Suljić
Date: 22.04.2024  
  
Under supervision of: Prof. Tobias Heed  


## Intro 

Lets say you are being touched - a buttefly lands on your hand. You may not notice, but actually your brain processes this touch differently, if you attend the touch or not.  

  <img src="/Methods/butterfly.jpg" alt="butterfly" width="100" height="100">

This is visible in electroencephalograms or also known as EEGs.  
Even more: a) your brain processes touch differently, if your hands are near or far away from each other and also b) if they are moving during the touch or not.  
All visible in the EEG (pretty cool, right?).  
  
We asked: is this also true for passively moved hands? Bonus: is this also true for continuously attended hands?  
  
  This was not done before, because EEG + movements is a very difficult thing to do. Normally, participants have to remain very still. We solved it via computational modelling.


**TLDR**; yes. We see differences between actively and passively moved hands in the EEG. Also, suprisingly, continously attended hands also show distance effects.  
 



## Methods

### Setup

Experimental Setup. We used the KINARM EndPoint Robot™  in combination with an EEG-system. A augmented reality display showed the experiment. An airsled accomplished frictionsless movements. Responses were made with a foot button.  
  
Figure (CC BY-NC-SA) adapted from Wood et al. (2019) under the CC BY-NC 4.0 licence.

![Setup](/Methods/Setup.png)


Figure (CC BY-NC-SA) showing the distances used in our experiment.

![nearVSfar](/Methods/nearfar.png)



### EEG Recording Examples
  
Example of an EEG recording from Frohlich et al (2016) under the CC BY-SA 4.0 licence.
  
![EEGrecording](/Methods/EEG_Example.png)
  
  

## Results
  
### Our EEG recording  

Our EEG distrubution over the brain after a tactile stimulus.
  
![Stimulus](/Results/TactileBeta.png)


### Checkups

  
We checked Responses (good):
  
![Responses](/Results/ResponseProb.png)

  
As well as Kinarm forces for active (good) and passive movements (also good):

![Forces](/Results/KinarmForces.png)

  
### Interference Signal
  
We noticed an interference signal in the data due to the motions. We excluded it via a linear deconvolution model:
  
  This way different events (button press, movements, experimental stimulation, etc.) can be isolated.

![Model](/Methods/LinearDeconvolution.png)

  
  One before (left) vs after (right) example. On the left picture there is a noticable "drift" in the solid lines (EEG traces).  
  After the extraction via the model there is no drift anymore and the lines are comparable!


### EEG over the Scalp

This is an example for the signals over the scalp. We grouped the EEG electrodes into 7 groups.

![ActiveEEG](/Results/ActiveMoving.png)
  
  
We also used control stimuli. They should show no systematic signal, which is the case:

![ActiveEmpty](/Results/ActiveEmpty.png)


### Hypthesis testing

We tested some hypotheses by cluster permutation testing: the EEG traces were compared to check for significant differences.
Here an example:  

![Permutation](/Results/C_Parietal.png)


### Active vs Passive

Here we see a schema of a part of the results: During active movements we see differences in near vs far stimulation (red diamonts), but not in the passive condition.  
This means the brain tracks the distances between the hands only for active movements!

![Schema](/Results/SchemaDistance.png)