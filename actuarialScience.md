# Actuarial Science

Chapter 8
- Insurance policies are often sold with a per-loss deductible of d
	- loss x <= d, insurance pays nothing
	- loss x > d, insurance pays x - d
1. Deductible
	1. Ordinary deductible: change a random variable into excess loss/left censored and shifted variable The difference depends on whether the result of applying the deductible is to be per payment or per loss, respectively
		- expected cost per loss: `E(Y^L) = E(X) - E(X ^ d)`
		- expected cost per payment: `E(Y^P) = E(Y^L) / (1 - F(d))`
	2. Franchise deductible: when loss x > deductible d, loss is paid in full. Modifies the ordinary deductible by adding the deductible when there is a positive amount paid
		- expected cost per loss: `E(Y^L) = E(X) - E(X ^ d) + d(1 - F(d))`
		- expected cost per payment: `E(Y^P) = E(Y^L) / (1 - F(d))`
2. Loss elimination ratio
	- The loss elimination ratio is the ratio of the decrease in the expected payment with an ordinary deductible to the expected payment without the deductible
	- `LER = E(X ^ d) / E(X)`
3. Inflation
	- For an ordinary deductible of d, after uniform inflation of 1 + r, the expected cost per loss is `(1+r){E(X) - E(X ^ d*)}`
	- `d* = d / (1+r)`
4. Policy limit
	- Opposite of deductible, right censored r.v.
	- loss < u, pay full loss
	- loss > u, pay u
	- Per loss and per payment is not relevant
	- For a policy limit of u, after uniform inflation of `1 + r`, the expected cost is: `(1+r) E(X ^ u/(1+r))`
5. Coinsurance
	- Insurance company pay `alpha` of the loss and policyholder pays the remaining.
	- `E(Y^L) = alpha x (1+r) x [E(X ^ u*) - E(X ^ d*)]`, `u* = u/(1+r)`, `d* = d/(1+r)`
	- `E(Y^P) = E(Y^L) / (1-Fx(d*))`
	- `E((Y^L)^2) = alpha^2 x (1+r)^2 x {E[(X ^ u*)^2] - E[(X ^ d*)^2]} - 2d*[E(X ^ u*) - E(X ^ d*)]`