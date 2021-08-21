+++
date = 2021-07-21T11:00:00Z
linktitle = " StudentLayer Part 1 - Introduction"
series = ["StudentLayer"]
title = " StudentLayer Part 1 - Introduction"
type = ["post", "posts  "]
weight = 1
[author]
name = "CJ Williams"

+++
I recently came across a dataset that I thought was cool - it was Hipo's
[university-domain-list](https://github.com/Hipo/university-domains-list). As a student, I wondered whether one could use this dataset to create a service that can perform student email verification, similar to [Unidays](https://myunidays.com) or [Studentbeans](https://studentbeans.com), to explore different technologies and concepts by adding new features.

So that's what I'm going to do. I'm going to create my own student verification service - _StudentLayer_. StudentLayer with be a testbed through which we can explore different topics as we build our service.

My initial language of choice will be Go. This choice is not for any particular reason other than having some experience in it, and I'd
like to try using it for something more complex.

Might language X be a better choice for this project? Probably. Am I bothered? Not really.

## The Not-So-Simple Problem

Our initial plan is to create a service that will take in an email and decide whether it is a 'real' student email.

How do we define 'real'?

For our purposes, we will define 'valid' as meeting the following criteria:

1. Is the domain part a valid educational institution domain? (i.e. ox.ac.uk - the domain for the University of Oxford)
2. Does the local part map to a real mailbox/account/user?

Our first criterion is somewhat unambiguous, as it is relatively easy to compile a list of educational domains. Higher
Countries generally recognise their Higher Education Institutions and provide them with a domain within the dedicated education namespace within the country's country code top-level domain (ccTLD).

The second part is trickier. How can we verify that the specific address (i.e. the local part) belongs to a student? We are probably not going to be able to distinguish a student from other university faculty. Perhaps some
universities format student emails differently from faculty emails, but this will vary significantly between universities and require significant effort.
(Machine learning could be the answer to this, but that's one for later)

Does this matter, though? Probably not. Students will, more often than not, significantly outnumber the number of faculty.
Furthermore, if we consider the business use case for our hypothetical service (i.e. discount codes), businesses will
probably be willing to accept some false positives. False negatives (i.e. not recognising a student) could mean a missed
opportunity to develop brand loyalty at such a crucial stage.

So, where does this leave us? If we can't prove that a university email belongs to a student, can we prove it belongs
to a human? In other words, do we need to make sure we haven't just been sent a random local part with a valid domain?

The traditional approach is with a verification email - a human (with access to the mailbox) must click on a link. This approach
works, but can we do it better? Can we remove the need for user interaction?

**I'll explore all of this in the following blog posts, so stay tuned as I build StudentLayer!**