# Gradient Decent:

1. The Algorithm used to train Linear Regression Model efficiently is known as *Gradient Descent Algorithm* .
2. Basically what this algorithm does is , it helps us find the minimum value of the cost function y by finding the steepest descent possible from the point of start.
3. If we were to calculate the minimum value of the cost function, we would keep changing the m and c in the cost function until we settle at or near minimum value.
4. Gradient Descent does this by finding out the path of steepest descent and reaching the local minima using that path.
5. A point to note in this algorithm is that if the point of origin is different in each case, the value and the position of the local minima may change as the path of the steepest descent may vary each time.

![[Gradient_Descent_Graphical_Representation.png|900x500]]

6. Moreover, gradient descent algorithm is also used to train some of the most advanced neural  network models also known as *Deep Learning Models*.
7. That being said Gradient Descent is a Widely used Algorithm in Machine Learning and Deep Learning.

	## Formula Involved in Gradient Descent : 

![[Squared_Error_Cost_Function.png]]

![[Gradient_descent_intro.png]]

- In the above figure, the value of the variables w and b from the cost function *J(w,b)=wx+b* are simultaneously updated by applying the above given formula until the values of w and b converge.
- After each simultaneous update of both the values w and b, the original values of w and b are updated until convergence.
- ![[Gradient_Descent_Code_Form.png]]
- Note that in the above figure the values of w and b are **first updated and then assigned back** for the next iteration.
- The symbol *alpha* denotes the currently used **Learning rate**  or the magnitude of the steps taken to climb down to the local minima. The Larger the value of the variable *alpha*, the more aggressive the gradient descent process and vice versa.
- The Final Formula that we will be using according to the [[Squared_Error_Cost_Function.png]] given by the current linear regression model is -
- ![[Gradient_Descent_Differentials.png]]
## Gradient Descent in Multiple Linear Regression:

1. In Multiple Linear Regression, there are multiple variable on which the linear regression model depends.
2. Eg. If there are *n* factors on which the price of the house depends, then the resulting linear regression model will be -

> f(w,x)= w1.x1 + w2.x2 + .........+ wn.xn +b

*Note: The w1.x1 here is the dot product of the given feature which is specially done by using np.dot(w,x) {import numpy as np}*
**w and x both are vectors in python**

Refer to [[Important funtions in Python]] for more about numpy in python.

Here are the equations you developed in the last lab on gradient descent for multiple variables.:
$$\begin{align*} \text{repeat}&\text{ until convergence:} \; \lbrace \newline\;
& w_j := w_j -  \alpha \frac{\partial J(\mathbf{w},b)}{\partial w_j} \tag{1}  \; & \text{for j = 0..n-1}\newline
&b\ \ := b -  \alpha \frac{\partial J(\mathbf{w},b)}{\partial b}  \newline \rbrace
\end{align*}$$
where, n is the number of features, parameters $w_j$,  $b$, are updated simultaneously and where  

$$
\begin{align}
\frac{\partial J(\mathbf{w},b)}{\partial w_j}  &= \frac{1}{m} \sum\limits_{i = 0}^{m-1} (f_{\mathbf{w},b}(\mathbf{x}^{(i)}) - y^{(i)})x_{j}^{(i)} \tag{2}  \\
\frac{\partial J(\mathbf{w},b)}{\partial b}  &= \frac{1}{m} \sum\limits_{i = 0}^{m-1} (f_{\mathbf{w},b}(\mathbf{x}^{(i)}) - y^{(i)}) \tag{3}
\end{align}
$$
* m is the number of training examples in the data set
*  $f_{\mathbf{w},b}(\mathbf{x}^{(i)})$ is the model's prediction, while $y^{(i)}$ is the target value
