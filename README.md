# BOOSTING IMPERCEPTIBILITY OF ADVERSARIAL ATTACKS FOR ENVIRONMENTAL SOUND CLASSIFICATION

This code is the submission for the anonymously submitted paper titled 'Boosting Imperceptibility of Adversarial Attacks for Environmental Sound Classification'.

![image](https://github.com/user-attachments/assets/b425bfde-3898-48a0-b4b5-c7c15a3d921f)

# FreqPsy Attack Algorithm

The conceptual blueprint of our proposed methodology is depicted in Figure~\ref{framework}. We will delve into its intricate components in the ensuing chapters. Initially, the process involves the creation of a nascent adversarial example. This is achieved by synergizing frequency-aware weighting with a gradient-based technique for adversarial attacks, a vital characteristic of the FPAA. The adversarial samples crafted in this phase are finely tuned to align with the frequency perception nuances of the human auditory system, embodying the psychoacoustic principles central to our method. Following this, the focus shifts to refining these adversarial examples, enhancing their stealth, and ensuring their auditory undetectability, as stipulated by the FPAA's design.

(1) Gradient Direction

In our approach, to streamline the computation process, the Short-Time Fourier Transform (STFT) representation of the original audio is employed as the basis for calculating the gradient direction, denoted as $\mathcal{D}$.

The methodology to ascertain $\mathcal{D}$ can be elucidated through the following mathematical expression:

$$\mathcal{D} = \mathrm{sign}\left(\nabla_x \mathcal{L}_{nn}(x, y)\right)$$

where $x$ represents the original spectral data, and $y$ signifies the actual label. The function $\mathcal{L}_{nn}(x,y)$ is the computational loss, and the application of the $\mathrm{sign}$ function is to determine the orientation of the gradient.

Our approach employs the human auditory threshold's A-weighting formula to ascertain frequency sensitivity. This method yields a set of weights reflecting the human ear's varying sensitivity across frequencies. We map these weights onto the perturbation range, denoted as $\mathcal{R}=[\varepsilon_{\text{min}}, \varepsilon_{\text{max}}]$, to determine their corresponding magnitudes. The formula below illustrates this process:

$$
\varepsilon(k) = M\left(\mathcal{A}(F[k]), d\right), \quad k=0,1,2,\dots,\left\lfloor\frac{N}{2}\right\rfloor
$$

where the $\mathcal{A}$ represents the A-weighting function, $F$ is the STFT, $k$ is the index within the spectrum bin, $M$ signifies the linear mapping function, and $d$ is a dispersion control factor. For example, Figure~\ref{imperceptibility} illustrates a scenario with an average perturbation size of 0.1 and a control factor of 0.2. In this illustration, the maximum perturbation value indicates the frequency of the least sensitivity, whereas the minimum value corresponds to the most sensitive frequency. To compute the perturbation step $\alpha(k)$ for each frequency in every iteration, we average the perturbations over $T$ iterations. We then perform an element-wise multiplication of $\mathcal{D}$ with $\varepsilon$ on each row, enabling the calculation of the cumulative perturbation $r$:

$$
r=\{r(k)| \delta(k) = \mathcal{D}(k)\times \frac{\varepsilon(k)}{T}, k=0,1,2,...,\left\lfloor\frac{N}{2}\right\rfloor\}
$$

The final step is to add $\delta$ to the original spectrum, as shown in the equation below:

$$
x_{t+1}=\Pi_{x+\mathcal{S}}\left(x_{t}+r\right)
$$

Here, $t$ denotes the iteration number, and $\Pi_{x+\mathcal{S}}$ is a projection operation that ensures the adversarial example falls within the range $x+\mathcal{S}$.

(2) Optimization with Masking Threshold

The optimization of the created perturbations $\delta$ culminates in the final psychoacoustic module, where we employ the following equation:

$$
\mathcal{L}_\theta(x, \delta)=\frac{1}{\left\lfloor\frac{N}{2}\right\rfloor+1}\sum _{k=0}^{\left\lfloor\frac{N}{2}\right\rfloor} \max (\{\bar{p} _\delta(k)-\theta_x(k), 0\})
$$

In this formula, $\bar{p}_\delta(k)$ is the normalized PSD estimation of the perturbation. The loss function $\mathcal{L}_\theta$ is designed to ensure that $\bar{p}_\delta(k)$ remains below the frequency masking threshold $\theta_x(k)$ of the original spectrum.

The comprehensive loss function that guides our optimization is composed of two distinct parts:

$$
\mathcal{L}(x, y,\delta)=-\mathcal{L}_ {nn}(x+\delta, y)+\alpha \cdot  \mathcal{L} _\theta(x+\delta, \delta)
$$



where $\alpha$ is a parameter that balances the importance of these two loss components, reconciling the trade-off between the effectiveness of the adversarial perturbation and its imperceptibility under psychoacoustic constraints. During the perturbation optimization process, if the generated adversarial samples are enough to make the model classify incorrectly, the weight alpha will increase. On the contrary, if the attack fails, the value will decrease. The optimization process is iterated a total of *I* times.



## Requirements

````
numpy==1.26.2
librosa==0.10.1
pystoi==0.3.3
pandas==1.5.3
sklearn==0.0.post1
scikit-learn==1.1.3
eagerpy==0.30.0
foolbox==3.3.3
tqdm==4.64.1
scipy==1.11.4
````

## Training Model

Train two models including VGG13 and VGG16. Please download the dataset from the following link and put it in the
`dataset` folder. 

[UrbanSound8K](https://www.kaggle.com/datasets/chrisfilo/urbansound8k)

[ESC-50](https://github.com/karolpiczak/ESC-50)

Below is an example.
```
cd FPAA
python train_model.py --dataset Urban8K --model VGG13 --batch_size 32 --num_epochs 100 --lr 0.0001
```

## Select a portion of the data that each model can predict successfully

We filter the ESC-50 dataset and store the results in the variable "esc_data". However, you need to filter the UrbanSound8K data set yourself. More specifically, in the UrbanSound8K dataset, please filter 50 data per category, while in the ESC-50 dataset, please select 10 data per category.


## Getting threshold
```
cd FPAA
python get_threshold.py --dataset Urban8K --psd_save_path urban_psd --sample_rate 22050 --window_size 2048
python get_threshold.py --dataset esc-50 --psd_save_path esc_psd --sample_rate 22050 --window_size 2048
```



## Generating adversarial examples

We use four attack algorithms to generate adversarial examples, including FGSM, BIM, PGD, and our proposed FPAA. The
following is an example. If you run the method we proposed, you need to specify the values of lr_stage, num_iter_stage,
and alpha. The other methods do not need to be specifically specified (because these parameters are the parameters used
in the method we proposed).

```
python run_attack.py --dataset Urban8K --model VGG13 --epsilon 0.1 --attack_method PGD_freq --save_path adv_example
python run_attack.py --dataset Urban8K --model VGG13 --lr_stage 0.001 --num_iter_stage 500 --epsilon 0.1 --attack_method PGD_freq_psy --save_path adv_example --alpha 0.07
```