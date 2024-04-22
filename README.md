# EEG Study Demo


Author: Kenan Suljić  
Date: 22.04.2024  
  
Under supervision of: Prof. Tobias Heed  


## Intro 

Lets say you are being touched - a buttefly lands on your hand. You may not notice, but actually your brain processes this touch differently, if you attend the touch or not.  

  <img src="/Methods/butterfly.jpg" alt="butterfly" height="500">

This is visible in electroencephalograms or also known as EEGs. 
   
Even more: a) your brain processes touch differently, if your hands are near or far away from each other and also b) if they are moving during the touch or not.  
  
All visible in the EEG (pretty cool, right?).  
  
  
We asked: 
- Is this also true for passively moved hands? 
- Bonus: is this also true for continuously attended hands?  
  

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
  
Example of an EEG recording (Andrii Cherninskyi licensed under CC BY-SA 4.0).  
  
  We used unsupervised machine learning to detect and clean up anomalies in our EEG recordings (example only).
  
![EEGrecording](/Methods/Unsupervised.png)
  
  

## Results
  
### Checkups


### Tactile stimulation 

EEG distrubution over the brain after a tactile stimulus (good).
  
![Stimulus](/Results/TactileBeta.png)



  
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


![beforevsafter](/Results/DeconvReconstruction.png)

### EEG over the Scalp

This is an example for the signals over the scalp. We grouped the EEG electrodes into 7 groups.

![ActiveEEG](/Results/ActiveMoving.png)
  
  
We also used control stimuli. They should show no systematic signal, which is the case:

![ActiveEmpty](/Results/ActiveEmpty.png)


### Hypthesis testing

We tested some hypotheses by cluster permutation testing: the EEG traces were compared to check for significant differences.
Here an example:  

![Permutation](/Results/C_Parietal.png)

  

  Example code for this figure:

  ´´´ python
    
    ymin, ymax = -2.8, 3.4
    palette = ['#1b1f22', '#970c0f', '#767a7f', '#f52b14']

    stim = 'single'
    con = 'passive'

    mov = 'moving_yes'


    if con == 'stationary':
        
        epochs_statio = {k: v for k, v in epochs.items() if (stim in k) & (con in k)}
        
        new_dict = {}
        
        for key, value in epochs_statio.items():
            # Split the key into components.
            components = key.split("/")

            # Remove the 'moving_no' or 'moving_yes' component.
            components.remove(components[3])

            # Join the remaining components back into a key string.
            new_key = "/".join(components)

            # If the new key is already in the new dictionary, append the value to the list of values.
            # Otherwise, start a new list of values.
            if new_key in new_dict:
                new_dict[new_key].append(value)
                new_dict[new_key] = mne.combine_evoked(new_dict[new_key], weights='nave')
            else:
                new_dict[new_key] = [value]
                
            epochs_sub = new_dict
        
        # for key, value in new_dict.items():
        #     epochs_sub = {}
        #     epochs_sub[key] = mne.combine_evoked(new_dict[key], weights='nave')
            
    else:
        epochs_sub = {k: v for k, v in epochs.items() if (stim in k) & (mov in k) & (con in k)}


    # contra
    picks = ['C5', 'C3', 'CP5', 'CP3']

    # ipsi
    # picks = ['C6', 'C4', 'CP6', 'CP4']
    # palette = ['#767a7f', '#1b1f22', '#970c0f', '#f52b14']



    fig = mne.viz.plot_compare_evokeds(epochs_sub,
                                picks=picks,
                                combine='mean',
                                colors={'attention_yes':'red', 'attention_no':'grey'},
                                linestyles={'stim_close':'solid', 'stim_far':'dotted'},
                                ylim=dict(eeg=(ymin, ymax)),
                                invert_y=True,
                                truncate_yaxis=False,
                                split_legend=False,
                                show_sensors='lower left',
                                time_unit='ms')

    fig[0].set_size_inches(10, 6)  # set new figure size

    fig[0].suptitle('Passive - Moving')
    # fig[0].tight_layout()

    axes = fig[0].axes

    axes[0].yaxis.grid(True)
    axes[0].yaxis.set_ticks(np.arange(ymin, ymax, 0.2), minor=True);
    axes[0].set_ylabel('Amplitude [μV]')

    axes[0].xaxis.set_ticks(np.arange(0, 350, 10), minor=True)
    axes[0].xaxis.set_ticks(np.arange(-100, 350, 50), minor=False)
    axes[0].set_xlabel('Time [ms]')

    for spine in axes[0].spines.values():
        spine.set_visible(True)

    axes[0].hlines(y=ymin,
            xmin=-100,
            xmax=350,
            colors='k')

    axes[0].vlines(x=0,
                ymin=ymin,
                ymax=ymax,
            colors='k')

    picks_str = ', '.join(picks)
    axes[0].set_title(picks_str)



    a = axes[0]
    lines = a.get_lines()
    legend = a.get_legend()
    # legend.set_title()

    for i, (line, legend_text) in enumerate(zip(lines, legend.get_texts())):
        line.set_color(palette[i])  # Change line color
        line.set_linewidth(2)
        
        # legend_text.set_text(hue_order[i])  # Change legend label
        # Create a new legend
        a.legend(lines, labels, loc='best')
        
    # Reorder the lines and labels
    lines = [lines[i] for i in order]
    # labels = [labels[i] for i in order]

        # legend_text.set_text(hue_order[i])  # Change legend label
    # Create a new legend
    a.legend(lines, labels, loc='best', title=hue)


    plt.savefig(path+'\\Plots\\Paper\\before and after unfold\\'+'Before_'+'-'.join(picks)+con+'_'+mov+'.svg', dpi=600, format='svg')
    plt.savefig(path+'\\Plots\\Paper\\before and after unfold\\'+'Before_'+'-'.join(picks)+con+'_'+mov+'.png', dpi=600, format='png')

  
  ´´´



### Active vs Passive

Here we see a schema of a part of the results: During active movements we see differences in near vs far stimulation (red diamonts), but not in the passive condition.  
This means the brain tracks the distances between the hands only for active movements!

![Schema](/Results/SchemaDistance.png)