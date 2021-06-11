# Computer Security

## Basic concepts
Security requirements: CIA paradigm
- Confidentiality: information can only be accessed by thos who are authorized
- Integrity: information can only be modified by authorized entities and only in the way such entities are entitled to modify them
- Availability: information must be available to all the parties that have the right to access it, within specified time constraint

Starting from here we can see that "A" conflicts with "C" and "I" and this is what makes security hard. I cannot encrypt the data and store them at the bottom of the ocean and call that secure, there are business requirements and time requirements that need to be satisfied that have become stricter and stricter over time.

### Definitions
- Vulnerability: a "shortcut" through the security process that undermines our assumptions that can potentially allow to violate the CIA paradigm.
- Exploit: a specific way to use one or more vulnerabilities to accomplish a specific objective that violates the constraint

There can be a vulnerability without having a working exploit.  
Also some vulnerabilities cannot be completely removed because it would be too inconvenient (impacting usability) or because they are too embedded in the system (hw vulnerabilites like spectre and meltdown). In those case we can only mitigate the vulnerability, we try to make it more difficult to exploit.

Security != protection -> it depends on the threat model we are defending against. We can only make comparisons in the same environment for the same type of threat.

- Threat modeling: defining what **assets** we are protecting against what **threats**
  - Assets: valuable things (yes, things, it can be anything: hw, sw, data, reputation,...) that we want to protect
  - Threats: who and in which ways can damage our assets
- Attack: *intentional* use of one or more exploits to violate the CIA properties of a target system
- Threat agent: whoever is carrying out the attack, also known as attacker

IMPORTANT NOTE:  
attacker != hacker  
an hacker is just someone with advanced understanding of computers and networks while an attakcer seeks to do damage and it's not necessarily knowledgable (see "cybercrime as a service" in DFC course)

- Risks: Evaluation of the probability of something bad happening to our assets and the damage that it would cause.

RISK = ASSETS x VULNERABILITIES x THREATS

The objective of security is to balance the reduction of the vulnerabilities and the damage containment versus the cost that this will bring to the organization (both direct like, management, operational and equipment and indirect like slower performance, less usability, less privacy).

ANOTHER NOTE:  
more money does not imply more security, we need to configure it appropriately and make sure that our users will not have to leave password on sticky notes under the monitor!

### Trust
We need to define boundaries, some part of the system that will be assumed to be secure (*trusted elements*). Those boundaries change depending on the threat model and on how much trouble/cost are we willing to go through to check everything. Examples of trusted elements for normal users can be the hw of the machine we are using or the operating system that is running. If you are a high profile target (like NSA) maybe you also want also to check those.  
Of course if an attacker is able to compromise something in our trusted elements then all our security assumptions will fall.

## Cryptography


## Authentication


## Introduction to software security


## Buffer overflows


## Format string bugs


## Web vulnerabilities


### Cross site scripting


### SQL injection


### Cross-site request forgery


## Network protocl attacks


## Secure network architectures


## Network security protocols


## Malicious software

## Appendix: x86 assembly crash course