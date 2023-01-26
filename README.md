# FQRS-detection

## 1 Introduction
Physicians track fetal growth and development during the perinatal period by monitoring
changes in of heart sounds, heart rate, and fetal electrocardiogram (ECG). The fetal ECG
reflects the electrophysiological activity of the fetal heart. The importance of a fetal ECG
waveform analysis is significant since it helps to detect fetal abnormalities during fetal
development, such as fetal distress and intrauterine hypoxia [2].
There are two approaches to obtaining a fetal ECG: invasive and non-invasive. The fetal
ECG can be registered directly with an electrode attached to the fetal head. The signal
then consists only of the fetal signals. It also can be obtained non-invasively with skin
electrodes on the maternal abdomen. This signal is much more complex since it also
contains the maternal QRS complexes, which are more visible.
The problem with the invasive ones is that we can only obtain such signals during labor.
Still, the ability to analyze fetal heart activity during pregnancy is no less important.

## 2 Problem Setting
The ultimate goal is to develop an algorithm that assumes a set of maternal abdominal
signals as an input parameter for a system and gets the locations of fetal QRS complexes
and a heart rate as an output.

## 3 Data Overview
The configuration of the abdominal electrodes comprised four electrodes placed around
the navel, a reference electrode placed above the pubic symphysis, and a common mode
reference electrode (with active-ground signal) placed on the left leg of women in labor,
between 38 and 41 weeks of gestation. Four different electrodes resulted in four signals
from the maternal abdomen. The acquisition of a direct fetal electrocardiogram was
carried out with a typical spiral electrode. All of the electrodes was guaranteed to remain
in the same position. [3]
As a result, there are five signals for each record:
- four maternal abdominal ECG,
- fetal ECG.
The last one is used as ground truth and reference to evaluate the results of the algorithm.

## 4 Methods
Non-invasive detection of fetal QRS complexes (FQRS) is a complex problem, but there
are several popular approaches to solving it. 

There could be a lot of steps to achieve the final goal. They can
vary from one pipeline to another, however, the key parts are mostly the same. That
includes
- Preprocessing:
_Principal Component Analysis (PCA), filtering the AECG, ..._
- Maternal component processing:
_detecting maternal R-peaks, FECG extraction by removing MECG from AECG sig-
nals, ..._
- Fetal R-peaks detection:
_FECG decomposition, FECG reconstructing, PCA, ..._

## 5 Algorithm analysis
### 5.1 Overview
The above-mentioned approaches to this problem surely have much more details of the
implementation. What I tried to do here is implement a simpler algorithm to solve the
same problem and analyze how it will do. The main steps with more detailed information
can be found below:
1. Choose least noised maternal ECG;
2. Filter chosen maternal ECG;
3. Subtract maternal signal from filtered;
4. Detect R-peaks in the output signal;
5. Calculate predicted fetal heart rate.
Since the abdominal ECG contains not only clear fetal signals but also maternal and some
additional noise, as a PCA I did the signal-to-noise ratio (SNR) calculation for each of
the AECGs. Assuming that ones with an SNR less than zero are quite noisy and can only
ruin the estimations – I leave them out of the further steps.
Next, the trick to extract FECG from AECG is to get the difference between two signals:
original AECG and filtered AECG (which is estimated to be the MECG).
To detect R-peaks locations, I calculate the location on each of the estimated FECG and
then use the weighted average of all of four or fewer result arrays with locations, where
the weights are proportional to the corresponding SNR value.
Now, having R-peaks locations it is easy to evaluate the frequency of contraction of the
heart muscle – which is the desired heart rate.

## 6 Experiments
### 6.1 Visualization

![Signals](img/signals.png?raw=true "Figure 1")

Figure 1 shows the 5-second snippet to show the initial data of the first record: four
AECGs and one FECG. The rest records were similar to it.
Next, the most interesting part of the algorithm is visualized in figure 2 below. The dark
blue line depicts one of the original maternal abdominal ECGs. The light blue one is
the estimate of FECG – the result of the difference between AECG and filtered AECG
(estimated MECG). Red markers show detected R-peaks locations.
The lower plot is given as a reference, and we can already see the pattern is clear. Esti-
mated fetal R-peaks coincide with the true ones.

![Prediction](img/prediction.png?raw=true "Figure 2")

And almost the last step for different records – we can see a snippet of original fetal
ECG along with the estimated R-peaks locations in Figure 3.


![Prediction](img/results.png?raw=true "Figure 3")

## 7 Pros & Cons
Taking into account the simplicity of the approach, it did a great job on the given records
of the dataset with a mean error of estimated heart rate of 4 bpm.
Yet, the dataset sample is quite small – 5 records are not enough to draw final conclusions.
Also, fortunately, all the children whose ECGs are in the dataset were healthy, that is, it
was not possible to see how the algorithm behaves with abnormal signals as an input.
Further improvements of the algorithm include trying to estimate not only the location
of the QRS complexes but also their amplitude since some of the abnormalities are more
related to the changes in amplitude and not the frequency.

## 8 Conclusions
In this project, we developed a simplified version of a fetal QRS-complexes detection
algorithm that takes four maternal abdominal electrocardiograms as input and gives the
calculated R-peaks locations and fetal heart rate as an output.
Test results support the claim that the method behaves well on the records without fetal
abnormalities and mostly gives a precious estimation of the heart rate. The advantages
and disadvantages of this approach were discussed along with the possible further im-
provement.
