# EEG Study Demo

Note: All figures under CC BY-NC-SA licence.


Author: Kenan Suljić  
Date: 22.04.2024  
  
Under supervision of: Prof. Tobias Heed  



## EEG Analysis and Tactile Spatial Attention

Imagine a butterfly landing gently on your hand—this simple touch triggers complex processes in your brain, which vary depending on your attention level. This is the research area on tactile attention.
  

  <img src="/Methods/butterfly.jpg" alt="butterfly" height="500">



Using electroencephalograms (EEGs), we observe how various factors influence the brain's response to touch:

- **Proximity of Hands**: Examines changes in brain processing when hands are close together versus far apart, a phenomenon known as tactile spatial attention.
- **Movement During Touch**: Analyzes differences in brain activity when hands are stationary versus moving.

#### Exploring Uncharted Territory in Tactile Spatial Attention

We delve into new aspects of tactile spatial attention by exploring:

- **Spatial Distribution of Tactile Attention**: Is tactile spatial attention coded gradually?
- **Continuous Attention**: What happens when hands are continuously attended during movement?
- **Passive Movement**: How does the brain respond when the touched hand is moved by external forces?

These questions are essential because integrating EEG with movement poses significant challenges; typically, EEG requires the subject to remain still. We've addressed these challenges through computational modeling.

#### Key Findings

**TL;DR**: Our research reveals distinct brain activity patterns for actively versus passively moved hands. For actively moved hands, we observed:
- **Proximity Effects**: Continuously attended hands also exhibit proximity effects, suggesting that:
  - There are either parallel distance effects alongside attentional effects, or
  - There is an attentional gradient that is enhanced when hands are closer together.

Interestingly, no distance effects were found in passive conditions, indicating that the brain either:
- Does not track the distance, or
- Does not enhance tactile attention during passive movements.



## Methods

### Setup

Experimental Setup. We used the KINARM EndPoint Robot™  in combination with an EEG-system. A augmented reality display showed the experiment. An airsled accomplished frictionsless movements. Responses were made with a foot button.  
  
Figure (Kenan Suljić under CC BY-NC-SA) adapted from Wood et al. (2019) under the CC BY-NC 4.0 licence.

![Setup](/Methods/Setup.png)


Figure (Kenan Suljić under CC BY-NC-SA) showing the distances used in our experiment.

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

  Figure reproduced from Dimigen & Ehinger (2021) under the CC BY-NC-ND 4.0 licence.

![Model](/Methods/LinearDeconvolution.png)

  
  One before (left) vs after (right) example. On the left picture there is a noticable "drift" in the solid lines (EEG traces).  
  After the extraction via the model there is no drift anymore and the lines are comparable!


![beforevsafter](/Results/DeconvReconstruction.png)  
  


  

  Example code for this figure:

  ```python
    # Define y-axis limits for EEG data visualization
ymin, ymax = -2.8, 3.4

# Define a color palette for plotting
palette = ['#1b1f22', '#970c0f', '#767a7f', '#f52b14']

# Set conditions and stimuli variables
stim = 'single'
con = 'active'
mov = 'moving_yes'

# Check if the condition is stationary
if con == 'stationary':
    # Filter epochs for the stationary condition including the specified stimulus
    epochs_statio = {k: v for k, v in epochs.items() if (stim in k) & (con in k)}
    
    # Initialize a new dictionary to store processed epochs
    new_dict = {}
    
    # Process each key, value pair in the filtered epochs
    for key, value in epochs_statio.items():
        # Split the key by '/' to manipulate its components
        components = key.split("/")

        # Remove the 'moving' component from the key
        components.remove(components[3])

        # Rejoin components into a new key
        new_key = "/".join(components)

        # Combine evoked responses or add new entries to the dictionary
        if new_key in new_dict:
            new_dict[new_key].append(value)
            new_dict[new_key] = mne.combine_evoked(new_dict[new_key], weights='nave')
        else:
            new_dict[new_key] = [value]
        
        # Store the modified dictionary back into epochs_sub
        epochs_sub = new_dict
    
else:
    # Filter epochs for non-stationary conditions including the specified stimulus and movement
    epochs_sub = {k: v for k, v in epochs.items() if (stim in k) & (mov in k) & (con in k)}

# Define electrode picks for plotting
picks = ['C5', 'C3', 'CP5', 'CP3']  # contra
# picks = ['C6', 'C4', 'CP6', 'CP4']  # ipsi (uncomment for ipsilateral picks)

# Plot compare evokeds with customized options
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

# Set figure dimensions
fig[0].set_size_inches(10, 6)

# Add a title to the figure
fig[0].suptitle('Active - Moving')

# Configure axes, labels, and grid
axes = fig[0].axes
axes[0].yaxis.grid(True)
axes[0].yaxis.set_ticks(np.arange(ymin, ymax, 0.2), minor=True)
axes[0].set_ylabel('Amplitude [μV]')
axes[0].xaxis.set_ticks(np.arange(0, 350, 10), minor=True)
axes[0].xaxis.set_ticks(np.arange(-100, 350, 50), minor=False)
axes[0].set_xlabel('Time [ms]')
for spine in axes[0].spines.values():
    spine.set_visible(True)
axes[0].hlines(y=ymin, xmin=-100, xmax=350, colors='k')
axes[0].vlines(x=0, ymin=ymin, ymax=ymax, colors='k')

# Display and adjust plot titles and legends
picks_str = ', '.join(picks)
axes[0].set_title(picks_str)
a = axes[0]
lines = a.get_lines()
legend = a.get_legend()
for i, (line, legend_text) in enumerate(zip(lines, legend.get_texts())):
    line.set_color(palette[i])  # Change line color based on the palette
    line.set_linewidth(2)
    # Update legend
    a.legend(lines, labels, loc='best')

# Save plots in SVG and PNG formats
plt.savefig(path+'\\Plots\\Paper\\before and after unfold\\'+'Before_'+'-'.join(picks)+con+'_'+mov+'.svg', dpi=600, format='svg')
plt.savefig(path+'\\Plots\\Paper\\before and after unfold\\'+'Before_'+'-'.join(picks)+con+'_'+mov+'.png', dpi=600, format='png')


  
  ```



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


# References

Dimigen, O., & Ehinger, B. V. (2021). Regression-based analysis of combined EEG and eye-tracking data: Theory and applications. Journal of vision, 21(1), 3-3.  
  
Wood, M. D., Khan, J., Lee, K. F., Maslove, D. M., Muscedere, J., Hunt, M., ... & Boyd, J. G. (2019). Assessing the relationship between near-infrared spectroscopy-derived regional cerebral oxygenation and neurological dysfunction in critically ill adults: a prospective observational multicentre protocol, on behalf of the Canadian critical care trials group. BMJ open, 9(6), e029189.
