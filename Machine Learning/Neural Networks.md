## Neural Network Intuition:

- The Neural Network in general mimics  the working of the human brains or more specifically how the neurons in the human brain works.
- According to the simple interpretation of the brain, it comprises of countless number of neurons that make up the biological neural network or the **brain** in general.
- The **Machine Learning** Algorithms mimic this biological principle to implement the *Neural Network* Technology that is now used today.
- The main idea regarding the working of neurons in that - a *neuron* takes up a set of data, processes it, makes some mathematical or logical computation about it and then passes on the resultant to a **new neuron** for further computation.

![[Pasted image 20221204171723.png]]

	As we can see in the above picture- input is passed on to the first neuron which gives back some output to the 
	next neuron and it continues to give back the final output.
***
## Need of Neural Networks:

- Due to the introduction of big data due to the rise of the internet, mobile phones and the digitization of the society, the classical Machine Learning Algorithms such as [[Logistic Regression Model]] and [[Linear Regression Model]] made it very difficult to increase the performance as more and more data is fed to it with passing time.
- If we were to train a small neural network on this same data set, the performance is likely to increase quite a bit as shown in the graph below.
- Therefore to deal with the increasing amount of data with utmost performance and accuracy, one must use a large neural network. Some of the good example of the applications of these large neural network implementations are speech and image recognition and natural language processing and many more.

![[Pasted image 20221204174134.png]]

***
## Neural Network Layers:

- The fundamental building block of most modern neural networks is *layer of neurons*. When we take these fundamental building block together, we will be able to build many large neural networks.
- Lets take and example to understand more about the Neural networks - 

![[Pasted image 20221204175713.png]]

- In the above example- The given **Neural Network** have 2 layers of neurons.
- The First layer of neurons have 3 neurons and this layer implements a simple [[Logistic Regression Model]] to give the probability values ranging from 0 to 1 in an array.
*Note: Layer 1 is also known as the hidden layer because its functionality is not visible directly to the outside user.*
![[Pasted image 20221204182010.png]]
- The second layer comprising of one neurons categorizes the activation values based on the set threshold values and gives forward a *yes* or *no* response.
- In the next example given below, an artificial neural network(ANN) is depicted - 

![[Pasted image 20230201115027.png]]

	The input layer (left, red), a hidden layer (in blue), and then the output layer (right, red).

- Assume this network is meant to predict the weather. The input values would be attributes relevant to weather such as time, humidity, air temperature, pressure, etc.
- These values would then be fed forward to the hidden layer while being manipulated by the weight values.
- Initially, weights are random unique values on every connection or synapse. The hidden layer’s new values are fed forward to the output, while being manipulated again by the weight values.
- Thus, this feeding the data forward by the current neuron layer to the next neuron layer for further computation is called **Forward Propagation** .
- At this point, it is important to recognise that the output would be completely random and incorrect. The manipulation during the feed-forward step contained no actual logic relevant to the problem because the weights start as random.
- However, we are training the ANN with a huge dataset containing previous weather forecasts with the same attributes and the result of these attributes (the target value).
- After the feed-forward stage, we can compare the incorrect output to the desired target value and calculate the error margin. Then we can back-propagate the network and adjust the weight values based on how they contributed to that error.
-  If we do this forward and back feeding 1,000 more times with each data item, the weights will start to manipulate future inputs in a relevant way. Oftentimes, even more success can come from training the same dataset multiple times.

	*The feed-forward step could be seen as guessing, and the back-propagation step educates that guess based on the margin of error. Over time, the guessing will become extremely accurate.*

***
## Important Packages Used in Developing Neural Network - 

### TensorFlow : 

- TensorFlow is an end-to-end open source platform for machine learning.
- TensorFlow is a rich system for managing all aspects of a machine learning system; however, this class focuses on using a particular TensorFlow API to develop and train machine learning models.
- TensorFlow APIs are arranged hierarchically, with the high-level APIs built on the low-level APIs.
- Machine learning researchers use the low-level APIs to create and explore new machine learning algorithms.
- In this class, you will use a high-level API named tf.keras to define and train machine learning models and to make predictions. tf.keras is the TensorFlow variant of the open-source [Keras](https://keras.io/) API.

![[Pasted image 20230201122551.png]]

### Keras :

-  *Keras* is a high-level, deep learning API developed by Google for implementing neural networks. It is written in Python and is used to make the implementation of neural networks easy. It also supports multiple back-end neural network computation.
- Keras is relatively easy to learn and work with because it provides a python frontend with a high level of abstraction while having the option of multiple back-ends for computation purposes. This makes Keras slower than other deep learning frameworks, but extremely beginner-friendly.
- Keras allows you to switch between different back ends. The frameworks supported by Keras are:
	-   [Tensorflow](https://www.simplilearn.com/tutorials/deep-learning-tutorial/tensorflow "Tensorflow")
	-   Theano
	-   PlaidML
	-   MXNet
	-   CNTK (Microsoft Cognitive Toolkit )

- Out of these five frameworks, TensorFlow has adopted Keras as its official high-level API. Keras is embedded in TensorFlow and can be used to perform deep learning fast as it provides inbuilt modules for all neural network computations. 
- At the same time, computation involving tensors, computation graphs, sessions, etc can be custom made using the Tensorflow Core API, which gives you total flexibility and control over your application and lets you implement your ideas in a relatively short time.
- ![[Pasted image 20230201124326.png]]
Reference Lab : [[Neurons and Layers]]
***
