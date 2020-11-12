# Speaker-identification-using-GMM

# Introduction 

In this study we will implement a text-independent speaker identification system based on GMM. The project is divided into three main parts: the training phase, the testing phase and the improvement phase.
There are a series of voice files from 16 different speakers in which a 16 kHz sampling frequency is used. Furthermore, these files are grouped into three different types: training files as “Training_List.txt” and testing files as “Test_List1.txt” and “Test_List2.txt”, with and without clean conditions, respectively.
In the training phase we will build the GMM models for each speaker using the appropriate files for that task and the EM algorithm. For the testing phase, the Maximum Likelihood criterion will be used, according to which the speaker of each test file will be recognized based on the GMM model that produces the highest log-likelihood sequence. This testing phase, is divided into two sections in which the performance of the algorithm will be evaluated in both clean and noisy conditions, using the corresponding audio files.
Finally, in the improvements phase, other features have been included and the number of Gaussians used to build the model have been varied, in order to increase the accuracy.
For the execution and evaluation of the project, a random seed of 1 was established when creating the models so that the comparisons in the accuracies for the different methods and parameters are consistent.

# Training phase

Once we establish the execution environment and the global variables that will be used as parameters for the methods, the audio files intended for the training phase are read. The objective of this phase will be to extract the MFCC features that characterize each audio file and build the GMM model for each file. For this, the bookstore library has been used, which facilitates the task with already defined functions.
The “lebrosa.load” function has been used to read the audio files and the “lebrosa.feature.mfcc” function has been used to extract the MFCC features for each file, in which the parameters previously defined for said function have been used. This function, for the particular case of "irm01_s01_test1.wav" which would be the first test file, would return 259 frames of 20 components in the form of a 20x259 matrix, which we will first have to transpose to get the appropriate dimensions and then use it in the construction of the GMM models.
Once the features for each file have been extracted, the GMM model is built using the “mixture.GasussianMixture” function. For each training file, its corresponding GMM model is saved since it will be used in the testing phase. In this first phase, 8 gaussians have been used to build the models and a diagonal covariance matrix.

# Testing phase

The task flow for both subsections will be the same:
   1. Extract the MFCC features using the same parameters as in the training phase.
   2. Calculate the log-likelihood for each model, for each testing file. In other words, for each testing file, the log-likelihood of the GMM models (calculated previously)           will be calculated. To calculate the log-likelihood of each model considering the features of each one, the function "score_sample" will be used as follows:                     "model.score_samples (MFCC)", where “model” refers to each previously calculated model, and MFCC are the features of the corresponding file.
   3. The corresponding speaker will be selected based on the GMM model that reports the highest log-likelihood value. Since the "score_samples" function specified above returns       a data series for each feature, a summation of these values will be made for each model in such a way that the score of each model in terms of log-likelihood is stored in       an array called “scores”. Then we select the largest of each of them to identify the speaker.
   
# Clean conditions   

The first thing is to read the audio files destined for this section and then repeat for each of them the three steps described above. When the different speakers identified by the method have been saved in a list, these speakers will be compared with the speakers that are stored in the text file corresponding to the clean condition audios. Following this method, an accuracy of 98.75% is obtained, which is a high value that indicates that our method is capable of correctly and accurately identifying audio file speakers in clean conditions.

# Noisy conditions

In this case the three steps specified at the beginning of this section have been repeated but, in this case, using the text file that refers to the audio files in noisy conditions. As expected, by including these noisy conditions, it will be more difficult for the method to correctly identify the speaker of each audio file, so the accuracy will be lower.
By evaluating the method under noisy conditions in the same way as under clean conditions, we obtain a low accuracy of 48.12%, which indicates that the method is not capable of identifying properly the speakers of the audio files.

# Improvements

The main objective of this section is to introduce improvements in the method of identifying speakers in such a way that the accuracy when evaluating said method is greater, especially trying to improve the accuracy and precision of the method in noisy conditions.
For this, three parameters have been fundamentally evaluated: the features that are extracted from each audio file, the iterations of the algorithm to build the GMM model and the number of Gaussians used.

  1. **Features:** studying the main features of an audio file and considering the ones that we can extract through the “lebrosa” library, we have decided to extract the              following features for each file:
  
     a. **Delta features:** delta features are the local estimate of the derivate of the input data. In this case, both the first and second derivatives are computed. For this,             the “lebrosa.feature.delta” method will be used, which receives as parameters the previously calculated MFCC features. As previously mentioned, taking the case of               the first test audio file, the MFCC features are presented in the form of a 259x20 matrix. When calculating the delta features since they are the local estimate                 derivatives of the MFCC data, they will have the same dimensions as this, obtaining as output a matrix of 259x20. Adding these features to the previously existing               MFCC, we obtain a new matrix of 259x40, including 20 new features in the columns.
         
      b. **Delta features, second derivates:** same procedure as with the first derivatives of the delta features, but in this case specifying order 2 in the function call.                  Likewise, with the first derivatives it returns a 259x20 matrix that will be added to the already existing features creating a new 259x60 features matrix.
       
      c. **Chroma features:** the chroma-based features are also often called pitch class profiles, they capture harmonic and melodic characteristics of the music, while                      remaining robust in pitch changes and instrumentation. To extract said features, the “lebrosa.feature.chroma_stft” method has been used. In this function, the same              parameters of n_fft and hop_length have been specified as when extracting the MFCC features in such a way that the same number of frames are obtained. In this case,              leaving the chroma number of features to obtain by default, we have as output a matrix of 259x12, that is, 12 features that will be added to the matrix of features              increasing its size to 259x72.
      
      d. **Zero crossing rate:** measure that quantifies the number of times in each interval that the signal amplitude passes through a value of zero. A high number of zero                  crossing implies that there is no dominant low-frequency oscillation. The method used is “lebrosa.feature.zero_crossing_rate” and the appropriate function                        parameters have been specified to return 259 frames including the zero-crossing rate of each one (259x1). With this output it is added to the set of features that                we had obtaining a new matrix of 259x73.
        
      e. **Fundamental frequency feature:** it is calculated by means of pitch, by performing a pitch tracking on thresholded parabolically-interpolated STFT through the                      "librosa.core.piptrack" method, which returns two data with the possible pitch values and the magnitudes of each one. That is why once this function is executed, we              will go through these values to select the pitch values that have a higher magnitude. Once again, when calling the function, the corresponding parameters have been              specified so the number of frames in this first audio that we are talking about is 259. Afterwards, we transform the array to a column matrix in such a way that we              have a matrix of 259x1 that will be added to our matrix of features getting a new matrix of 259x74.
        
      f. **Spectral-roll off feature:** measure of how much right-skewedness of the power spectrum, measured in fraction of bins in the power spectrum at with 85% of the power                is at lower frequencies. The higher the values, the more right-skewed spectra. The function executed for this is “booksa.feature.spectral_rolloff” in which the                  parameters have been specified so it returns 259 frames in the case of the first audio. The function therefore returns 259 frames in rows and a column with the                  spectral-roll off, which will be added to the features matrix having a new and final matrix of dimensions 259x75.
         
  2. **Number of iterations:** the number of iterations was increased to a maximum of 500 in order to ensure that the “mixture.GaussianMixture” function always converged. This            avoids the warning that sometimes appeared indicating that the function did not converge. This was because the likelihood increase of GMM training is still bigger than          the threshold of sklearn default setup when the final iteration of GMM training.
    
  3. **Number of gaussians:** initially, it started with 8 gaussians, which considering the changes and improvements described above, reported an accuracy at the time of                  evaluating the method of 77.50%. It should be noted that these improvements are being carried out in noisy conditions. Once having said reference measure, different              values of gaussians were tested:
  
     | Gaussians | Accuracy |
     |-----------|----------|
     |     2     |  70,62%  |
     |     20    |  80,62%  |
     |     70    |  67,50%  |
     |     120   |  70,00%  | 
     |     170   |  57,50%  |

   As we can see, there seems to be a peak in which the accuracy as a function of the number of Gaussians is greater, which is located around 20. From which, as much as if we      increase or decrease the Gaussian number, this accuracy decreases. Therefore, if we had to choose a value for the number of Gaussians, it should be around 20, since it is the    value that reports the higher accuracy.
   
   
   
We can conclude that we have managed to increase the accuracy of the method in noisy conditions considerably with the improvements. The results show that the accuracy in noisy conditions was almost doubled, achieving an acceptable value by identifying speakers in the method.
We realized there are still more features and variables that could have been considered and included, but it was decided to include those that are most relevant and that could be extracted through the "lebrosa" library. We consider this practice to be an extremely useful and practical application in the treatment and recognition of sounds.
