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
