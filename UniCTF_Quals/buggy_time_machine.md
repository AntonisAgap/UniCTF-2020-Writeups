# Buggy Time Machine  HTB x Uni CTF 2020 - Quals 

***The creator provided us with a docker to connect to the Flask web app and its source code: 
public.py.***

Lets check out the code:

```python
class TimeMachineCore:
	
	n = ...
	m = ...
	c = ...
		

	def __init__(self, seed):
		self.year = seed 

	def next(self):
		self.year = (self.year * self.m + self.c) % self.n
		return self.year

app = Flask(__name__)
a = datetime.now()
seed = int(a.strftime('%Y%m%d')) <<1337 % random.getrandbits(128)
gen = TimeMachineCore(seed)


@app.route('/next_year')
def next_year():
	return json.dumps({'year':str(gen.next())})

@app.route('/predict_year', methods = ['POST'])
def predict_year():
	
	prediction = request.json['year']
	try:

		if prediction ==gen.next():
			return json.dumps({'msg': msg})
		else:
			return json.dumps({'fail': 'wrong year'})

	except:

		return json.dumps({'error': 'year not found in keys.'})

@app.route('/travelTo2020', methods = ['POST'])
def travelTo2020():
	seed = request.json['seed']
	gen = TimeMachineCore(seed)
	for i in range(hops): state = gen.next()
	if state == 2020:
		return json.dumps({'flag': flag})

@app.route('/')
def home():
	return render_template('index.html')
if __name__ == '__main__':
	app.run(debug=True)
```
We can see the class TimeMachineCore which contains three unknown variables ***(n,m,c)***. The class has two methods

- The initiliazation method ***__init__*** which takes as an argument a seed and uses it as a year.

- The second method calculates a value which comes from this expression: 

<a href="https://www.codecogs.com/eqnedit.php?latex=S_{i&plus;1}&space;=&space;(S_{i}*a&plus;c)mod&space;n" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{i&plus;1}&space;=&space;(S_{i}*a&plus;c)mod&space;n" title="S_{i+1} = (S_{i}*a+c)mod n" /></a>

***Hmm...Interesting.***

After some research we found that this is actually a Linear Congruential Random Number Generator. 

A really good article which we based our code in for this challenge and understand LCGs is this: https://tailcall.net/blog/cracking-randomness-lcgs/

Moving forward we see that the Flask app contains four endpoints:
- ***/***, which shows us the homepage of the web app
- ***/next_year***, which calculates the next state of the LCG and returns us the value
- ***/predict_year***, in which, if your predict the next state of the LCG it returns a message
- ***/travelTo2020***, in which, if you give an initial state which after *hops* states returns the state = 2020 you get the flag.



Ok. Lets check the last part.

So the equation to find what is the next state is:

<a href="https://www.codecogs.com/eqnedit.php?latex=S_{hops&plus;1}&space;=&space;S_{hops}*m&plus;c&space;modn&space;(1)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{hops&plus;1}&space;=&space;S_{hops}*m&plus;c&space;modn&space;(1)" title="S_{hops+1} = S_{hops}*m+c modn (1)" /></a>

We also have:

<a href="https://www.codecogs.com/eqnedit.php?latex=S_{hops&plus;1}&space;=&space;2020&space;(2)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{hops&plus;1}&space;=&space;2020&space;(2)" title="S_{hops+1} = 2020,(2)" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=S_{hops}&space;=&space;S_{0}&space;*&space;m^(hops-1)&plus;&space;c*m^(hops-2)&space;&plus;&space;c,(3)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{hops}&space;=&space;S_{0}&space;*&space;m^(hops-1)&plus;&space;c*m^(hops-2)&space;&plus;&space;c,(3)" title="S_{hops} = S_{0} * m^(hops-1)+ c*m^(hops-2) + c,(3)" /></a>

If we replace *(2)* and *(3)* in *(1)* we get the following linear congruence:

<a href="https://www.codecogs.com/eqnedit.php?latex=S_{0}&space;*&space;m^(hops)&plus;&space;c*m^(hops-1)&space;&plus;&space;c&space;=&space;2020modn" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{0}&space;*&space;m^(hops)&plus;&space;c*m^(hops-1)&space;&plus;&space;c&space;=&space;2020modn" title="S_{0} * m^(hops)+ c*m^(hops-1) + c = 2020modn" /></a>

So if we want to solve for S0 (initial state) we need to find these variables:
- *hops*
- *m*
- *c*
- *n*

Alright, lets go crack this generator to find *(m,n,c)*


## Cracking LCGs
Because all PNRGs (Pseudo Random Number Generators) are deterministic,knowing a full internal state of a PNRG allows an attacker to predict all future and previous states.
θθ
Linear Congruential Generators are defined by three integers:

- A multiplier
- An increment
- A modulus

From the code above we can see that the multiplier is the value m,the increments is c and the modulus is n.
All three values are unknown to us (the attacker) in this senario.

### Cracking the modulus

To find the modulus we can use a number theory trick: if we have a few random multiples of n, there is a large probability that their gcd will be equal to n.
We need to take a gcd from values where:

<a href="https://www.codecogs.com/eqnedit.php?latex=X=0modn\Rightarrow&space;X=k*n,for\,&space;X!=0" target="_blank"><img src="https://latex.codecogs.com/gif.latex?X=0modn\Rightarrow&space;X=k*n,for\,&space;X!=0" title="X=0modn\Rightarrow X=k*n,for\, X!=0" /></a>

To do this we introduce the sequence:

<a href="https://www.codecogs.com/eqnedit.php?latex=T_{n}&space;=&space;S_{n-1}-&space;S_{n}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?T_{n}&space;=&space;S_{n-1}-&space;S_{n}" title="T_{n} = S_{n-1}- S_{n}" /></a>

Let's say we capture 5 states of the LCG ***S0,S1,S2,S3,S4*** then we have:

<a href="https://www.codecogs.com/eqnedit.php?latex=T_{0}&space;=&space;S_{1}&space;-&space;S_{0}\,&space;(1)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?T_{0}&space;=&space;S_{1}&space;-&space;S_{0}\,&space;(1)" title="T_{0} = S_{1} - S_{0}\, (1)" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=T_{1}&space;=&space;S_{2}-S_{1}&space;=&space;(S_{1}*m&plus;c)&space;-&space;(S_{0}*m&plus;c)&space;=&space;m*T_{0}(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?T_{1}&space;=&space;S_{2}-S_{1}&space;=&space;(S_{1}*m&plus;c)&space;-&space;(S_{0}*m&plus;c)&space;=&space;m*T_{0}(modn)" title="T_{1} = S_{2}-S_{1} = (S_{1}*m+c) - (S_{0}*m+c) = m*T_{0}(modn)" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=T_{2}&space;=&space;S_{3}-S_{2}&space;=&space;(S_{2}*m&plus;c)&space;-&space;(S_{1}*m&plus;c)&space;=&space;m*T_{1}(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?T_{2}&space;=&space;S_{3}-S_{2}&space;=&space;(S_{2}*m&plus;c)&space;-&space;(S_{1}*m&plus;c)&space;=&space;m*T_{1}(modn)" title="T_{2} = S_{3}-S_{2} = (S_{2}*m+c) - (S_{1}*m+c) = m*T_{1}(modn)" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=T_{3}&space;=&space;S_{4}-S_{3}&space;=&space;(S_{3}*m&plus;c)&space;-&space;(S_{2}*m&plus;c)&space;=&space;m*T_{2}(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?T_{3}&space;=&space;S_{4}-S_{3}&space;=&space;(S_{3}*m&plus;c)&space;-&space;(S_{2}*m&plus;c)&space;=&space;m*T_{2}(modn)" title="T_{3} = S_{4}-S_{3} = (S_{3}*m+c) - (S_{2}*m+c) = m*T_{2}(modn)" /></a>

Now let's generate our desired operation:

<a href="https://www.codecogs.com/eqnedit.php?latex=T_{2}*T_{0}&space;-&space;T_{1}*T_{1}&space;=&space;(m*m*T_{0}*T_{0})-(m*T_{0}*m*T_{0})&space;=&space;0(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?T_{2}*T_{0}&space;-&space;T_{1}*T_{1}&space;=&space;(m*m*T_{0}*T_{0})-(m*T_{0}*m*T_{0})&space;=&space;0(modn)" title="T_{2}*T_{0} - T_{1}*T_{1} = (m*m*T_{0}*T_{0})-(m*T_{0}*m*T_{0}) = 0(modn)" /></a>

We can generate few values congruent to 0 such as this, and crack LCG with mentioned "trick".

 Let's write that in Python:

 ```Python
 def crack_unknown_modulus(states):
    print(". Cracking modulus")
    diffs = [s1 - s0 for s0, s1 in zip(states, states[1:])]
    zeroes = [t2*t0 - t1*t1 for t0, t1, t2 in zip(diffs, diffs[1:], diffs[2:])]
    modulus = abs(reduce(gcd, zeroes))
```

### Cracking the multiplier

Now that we have the modulus we can find the multiplier using three states of the LCG that we can transform in two linear equations with two unknowns as such:

<a href="https://www.codecogs.com/eqnedit.php?latex=S_{1}&space;=&space;S_{0}*m&plus;c(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{1}&space;=&space;S_{0}*m&plus;c(modn)" title="S_{1} = S_{0}*m+c(modn)" /></a>
<a href="https://www.codecogs.com/eqnedit.php?latex=S_{2}&space;=&space;S_{1}*m&plus;c(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{2}&space;=&space;S_{1}*m&plus;c(modn)" title="S_{2} = S_{1}*m+c(modn)" /></a>
<a href="https://www.codecogs.com/eqnedit.php?latex=S_{2}-S_{1}=&space;S_{1}*m-S_{0}*m(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{2}-S_{1}=&space;S_{1}*m-S_{0}*m(modn)" title="S_{2}-S_{1}= S_{1}*m-S_{0}*m(modn)" /></a>
<a href="https://www.codecogs.com/eqnedit.php?latex=m&space;=&space;(S_{2}-S_{1})/(S_{1}-S_{0})(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?m&space;=&space;(S_{2}-S_{1})/(S_{1}-S_{0})(modn)" title="m = (S_{2}-S_{1})/(S_{1}-S_{0})(modn)" /></a>

In Python:
```python
def crack_unknown_multiplier(states, modulus):
    multiplier = (states[2] - states[1]) * modinv(states[1] - states[0], modulus) % modulus
```
### Cracking the increment

Now that we know the modulus and the multiplier we can find the increment using simple linear equations:'

<a href="https://www.codecogs.com/eqnedit.php?latex=S_{1}&space;=&space;S_{0}*m&plus;c(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{1}&space;=&space;S_{0}*m&plus;c(modn)" title="S_{1} = S_{0}*m+c(modn)" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=c&space;=&space;S_{1}-S_{0}(modn)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?c&space;=&space;S_{1}-S_{0}(modn)" title="c = S_{1}-S_{0}(modn)" /></a>

Let's implement that in Python:
```Python
def crack_unknown_increment(states, modulus, multiplier):
    increment = (states[1] - states[0]*multiplier) % modulus
```

```
Modulus = 2147483647
Multi = 48271
Incre = 0
```


Now that we have the internal state of the LCG, we can finally predict a next state to find what the message says.
```python
# find message
print(". Predicting next year to get hops")
next_state = (int(state) * mult + inc) % modulus
req = {
    "year": next_state
}
r = requests.post(url = url+"predict_year", json=req)
resp = r.json()
print(resp)
```

Hmm.. It seems that the message gives us the number of hops.
```python
hops = 876578
```



## Finding the flag

Perfect! Now that we have the internal state and the hops we can solve the previous linear congruence for ***S0*** using Euler's theorem:

<a href="https://www.codecogs.com/eqnedit.php?latex=S_{0}&space;*&space;m^(hops)=&space;2020modn" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_{0}&space;*&space;m^(hops)=&space;2020modn" title="S_{0} * m^(hops)= 2020modn" /></a>

```python
def linear_congruence(a, b, m):
    if b == 0:
        return 0

    if a < 0:
        a = -a
        b = -b

    b %= m
    while a > m:
        a -= m

    return (m * linear_congruence(m, -b, a) + b) // a
```

## Full Code:

payload.py
```python
import requests
import json
from congsolver import linear_congruence


url = "http://docker.hackthebox.eu:30451/"
# getting states
print(". Getting states")
states = []
for i in range(7):
    r = requests.get(url = url+"next_year")
    data = r.json()
    req = {
        "year": data["year"]
    }
    year = str(req['year'])
    states.append(int(year))


#s1 = s0*m + c (modn)
#s2 = s1*m + c (modn)
#s3 = s2*m + c (modn)

# s1 - (s0*m + c) = k_1 * n
# s2 - (s1*m + c) = k_2 * n
# s3 - (s2*m + c) = k_3 * n


import math
import functools

reduce = functools.reduce
gcd = math.gcd

def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, x, y = egcd(b % a, a)
        return (g, y - (b // a) * x, x)

def modinv(b, n):
    g, x, _ = egcd(b, n)
    if g == 1:
        return x % n

def crack_unknown_increment(states, modulus, multiplier):
    print(". Cracking increment")
    increment = (states[1] - states[0]*multiplier) % modulus
    return modulus, multiplier, increment

def crack_unknown_multiplier(states, modulus):
    (". Cracking multiplier")
    multiplier = (states[2] - states[1]) * modinv(states[1] - states[0], modulus) % modulus
    return crack_unknown_increment(states, modulus, multiplier)

def crack_unknown_modulus(states):
    print(". Cracking modulus")
    diffs = [s1 - s0 for s0, s1 in zip(states, states[1:])]
    zeroes = [t2*t0 - t1*t1 for t0, t1, t2 in zip(diffs, diffs[1:], diffs[2:])]
    modulus = abs(reduce(gcd, zeroes))
    return crack_unknown_multiplier(states, modulus)


modulus,mult,inc = crack_unknown_modulus([states[0],states[1],states[2],states[3],states[4],states[5],states[6]])
print("Modulus",modulus)
print("Multi",mult)
print("Incre",inc)

r = requests.get(url = url+"next_year")
data = r.json()
req = {
    "year": data["year"]
}
data = r.json()
state = str(req['year'])

# find message
print(". Predicting next year to get hops")
next_state = (int(state) * mult + inc) % modulus
req = {
    "year": next_state
}
r = requests.post(url = url+"predict_year", json=req)
resp = r.json()
print(resp)

hops = 876578

print(". Solving linear congruence ")
#  mult^hops*x=2020%modulus"
seed = linear_congruence(pow(mult,hops,modulus),2020,modulus)
req = {
    "seed" : seed
}
r = requests.post(url=url+"travelTo2020",json=req)
print(r.text)
```
congsolver.py
```python
from math import gcd
def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, x, y = egcd(b % a, a)
        return (g, y - (b // a) * x, x)

def modinv(b, n):
    g, x, _ = egcd(b, n)
    if g == 1:
        return x % n


def linear_congruence(a, b, m):
    if b == 0:
        return 0

    if a < 0:
        a = -a
        b = -b

    b %= m
    while a > m:
        a -= m

    return (m * linear_congruence(m, -b, a) + b) // a
```

After all that we can finally get the flag:
```
HTB{l1n34r_c0n9ru3nc35_4nd_prn91Zz}