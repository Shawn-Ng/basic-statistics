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