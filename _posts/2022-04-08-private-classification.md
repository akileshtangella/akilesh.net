---
layout: post
title:  "Private Classification"
tags: 
    - privacy
    - machine learning
    - zero knowledge proofs
    - blockchains
usemathjax: true
---
## Setup
Suppose Alice owns a health insurance company which has developed a logistic regression model with weights $$w = (w_1,...,w_d)$$ for checking whether a person qualifies for insurance. Bob wants to check whether he is eligible for insurance at Alice's company, but does not want to reveal his personal medical data $$x = (x_1,...,x_d)$$. In this article, we discuss how to use well-known cryptographic primitives to solve this problem.

## The Function to Evaluate 
Suppose $$w \cdot x > 0$$ (where $$\cdot$$ is the dot product between $$w$$ and $$x$$) corresponds to Bob qualifying for health insurance and $$w\cdot x \leq 0$$ corresponds to Bob not qualifying for health insurance. Alice and Bob want to compute the function $$f(w \cdot x)$$. $$f(z)$$ is the function which evaluates to $$1$$ if $$z > 0$$ and $$0$$ if $$z \leq 0$$.

## Case 1: Alice's Model is Not Proprietary
If Bob knows Alice's model weights but does not want to reveal $$x$$, he can submit a zero-knowledge proof that he knows an $x$ such that $$f(w \cdot x) = 1$$, which Alice can then verify. 

### Stopping Bob from Fraudulent Behavior
What stops Bob from creating a zero-knowledge proof with respect to some arbitrary $$x$$ instead of his actual medical data. This is where the blockchain comes in. Bob can have his doctor sign $$h(x)$$ where $$h$$ is a cryptographically secure hash function. Either Bob or his doctor can then post this signature on-chain. Alice then verifies this signed value as $$h(x)$$ and $$h(x)$$ serves as a public input to the zero-knowledge proof. In particular, Bob submits a zero-knowledge proof:
- That he knows the pre-image $$x$$ of $$h(x)$$
- That $$f(w \cdot x) = 1$$

### Stopping Alice from Fraudulent Behavior
Suppose Alice secretly hates Bob and even though his zero-knowledge proof indicates that he should qualify for health insurance, Alice rejects him. Again for this issue, we can use blockchains. We can require that Alice verify Bob's proof in a smart contract so its evaluation is public. Then if Alice doesn't follow the results of the verification she can be unequivocally blamed.

## Case 2: Alice's Model is Proprietary
In this case Alice and Bob want to evaluate $$f(w \cdot x)$$ with Alice not revealing $$w$$ and Bob not revealing $$x$$. To do this, we have to make use of a cryptographic primitive known as secure two-party computation, which allows two parties to jointly compute a function without revealing their private inputs. We will cover the mathematics behind secure two-party computation in a later blog post.


