# Digital Forensics and Cybercrime
**Risk**: statistical and economical evaluation of the exposure to damage because of the presence of vulnerabilities and threats.

RISK = ASSETS x VULNERABILITIES x THREATS

Threats are not under our control, we need to understand their nature in order to create better defence against them.
### Threat landscape
We can classify threats according to different categories:
- internal vs external
- generic vs targeted
- financially motivated vs other motivations

Financially motivated threats are the most common and i a way easier to deal with because we need to make an attack not economically convenient to prevent them. Other motivations may be more difficult to predict and can include a variety of different actors from a grudged employee to a state sponsored attack.

While for a single attack the threat that can potentially deal more damage is an internal specific threat, for society as a whole the external generic are the one that cause more damage because in the end it's a numbers game (for examples, social engineering attacks)
>The attackers' job is to make one person in the company to click on one link. The security's job is to get every person in the company not to click on any link.

We need to understand that not all breaches can be prevented and we need to have a way to detect them fast and have an incident response strategy to deal with them.

### Cybercrime industry
In the beginning, during the '90 malware was made mainly for fun and demostration purposes or to show off one's capabilities. Malware produced for financial gain started to appear at the beginning of the 2000 when normal people started to connect to the internet and had valuable data to protect on their machines. This became a mass issue and criminals became more and more organized. Later during the 2010 years malware started to be used also in military operation and state sponsored espionnage or attacks against infrastructures.
##### APT
Advanced persistent threats that aim at taking control of infrastructures and planting backdoors to maintain access or repeatedly break into systems. Often associated with states.

### Financially motivated attacks
Attackers are interested in monetizing their attacks. Monetization can be:
- direct: stolen credit cards, selling fake AV, ransomware(surging in recent years)
- indirect: rent botnet infrastructure, collect data, abuse of computing resources

Mainly two categories of malware are employed:
- credential stealers, in particular banking trojans
- RAT, remote access tools, to take control of machines

Those malware are delivered mainly through email attachments or drive-by download from malicious websites. These activities are managed by different groups of people that provide services to enable cybercrime for instance selling exploits kits, building malware (CYBERCRIME PRODUCERS) rent infrastructure, money laundering, etc... (CYBERCRIME ENABLERS)

#### Money mules
How do criminals extract money from electronic accounts? As long as it is in the banking world there are two issues: it can be recalled and it can be tracked.
Common ways to launder money is:
- withdraw cash using stolen card (inconvenient)
- someone who willingly act as a money mule for you allowing to withdraw money
- directly buy goods with the stolen money or crypto (see section about crypto later)
- variants of the "nigerian prince scam"

The latter is used widely and works like this:
- convince someone to let the scammer deposit money on their account
- ask them to withdraw 90% and let them keep the other 10%
- transfer the 90% to the scammer via a wire transfer without passing through banks (MoneyGram, Western Union,...)
to an office in some suitable place with no controls
- the person worst case is being arrested by the police for stealing.

The trick here is to make people feel "smart" and they are cheating the system and taking advantage from it.
>Being evil feels good sometimes

### Cryptocurrencies in cybercrime
We consider bitcoin for simplicity.
Usages:
- ransomware
- money laundering
- black market payments

What are the advantages?
#### Bitcoin is pseudo-anonymous  
Each entity can create as many keys as he wants and they are not directly correlated to him. There are anyways things to look for because since the whole history of transaction is public they can be analyzed and correlate different addresses togheter to get the tree of entities involved (we still don't know who is behind an address but we knwo that several addresses belong to the same person/entity)  
example:
- multiple inputs to a transaction
- new address generated to gather the unspent amount of a transaction

By classifing addresses by owner we can get some information about the owner:
- did it get BTC through mining?
- was it used on an exchange?
- was it mentioned on some forum?
- was it used in a scam?

Further challenges are presented by other currencies that are built to achieve more anonymity like Monero.

## Introduction to Digital Forensics
FORENSICS: application of scientific analysis methods to reconstruct evidence. An evidence is a proof of something that will be used in a tribunal.

DIGITAL FORENSICS: forensics applied to digital data, computers, network data.

Court cases only care about demonstrable facts and evidence is subject to regulation about how it can be acquired, when it is valid. Regulation may change under different legislations.
For example in the USA there is the concept of chain of custody that can determine whether a piece of evidence can be shown to the jury or not while in Italy all evidence can be shown to the judge even if inadmissible (in this case it can't be used in the motivation for the conviction). It's up to the judge to evaluate proofs.

#### Daubert standard in US
EXPERT WITNESS: expert qualified by knowledge, skill, experience, training or education.  
An expert witness may testify if:
- his scientific, technical or other specialized knowledge will help the trier of fact to understande the evidence or to determine a fact in issue
- the testimony is based on sufficient facts or data
- the testimony is the product of reliable principles and methods
- the expert has reliably applied those principles and methods to the facts of the case

The last 3 points define that the testimony have to be scientific.

#### What do we mean by scientific?
There are two main requirements:
- REPEATABILITY
    - the experiment can be recreated by someone
    - a description of the experiment is sufficient to assess is validity
- FALSIFIABILITY
    - is there an experiment that can prove the contrary?

Factors to consider in court:
- is the technique generally accepted in the scientific community?
- has it been subjected to peer review and publication?
- can be and has been tested?
- is the known/potential error rate acceptable?
- was the technique/research conducted outside of the specific litigation? (can be biased)

## Four phases of investigation
1. source acquisition
2. evidence identification
3. evaluation
4. presentation

We will deal with them in order.  

### Source acquisition
NOTE: bear in mind that these techniques were developed in the US with the US legal framework in mind.  
The main point is that digital evidence is brittle. If it is modified there is no way to tell (not tamper evident) and theoretically it can also be created (a perfect fake)  
For these reason we need at least to ensure that when we collect digital evidence we are not modifying it in the process and no modifications occur after the acquisition.  
This is done using hashes. 
Hashes are used as a "digital seal" on the evidence to make sure that it has not been modified **since the moment the hash was calculated.** This is very important because it doesn't mean that it was not modified before, also the hash by itself cannot help us tell what was modified.
Hashes should alse be kept correctly, preferrably sealed in writing or digitally signed.

To acquire data often linux based tools are used or embedded devices that perform the copy. The copy that we want is a bit by bit copy of the whole drive (bitstream image) in order not to loose any information that may be present

General approach:
- disconnect media from orignal machine if possible
- use a write blocker to ensure no modification
- compute the hash of the source  
`dd if=/dev/sda conv=noerror,sync | sha256su`
- copy the source bit by bit  
`dd if=/dev/sda of=/tmp/acquisition.img conv=noerror,sync` 
- compute hash of the source and copy to compare them (yes we need to recompute the original to ensure no modification happened)  
`dd if=/dev/sda conv=noerror,sync | sha256sum`
`sha256sum /tmp/acquisition.img`
It can be good to also compute SHA-1 and MD5 hashes for redundancy

Challenges:
- TIME
  - modern drives are huge (several TB) but transfer speeds are slow (100-150 MB/s) so these processes take a lot of time (we can automate something but still takes a long time, see dcfldd that hashes and copies at the same time)
- SIZE
  - for a large scale investigation there can be hundreds of terabytes of data that require to have a NAS/SAN since using external drives is impractical. We also need a way to move data around
- ENCRYPTION
  - unfeasible to get the data without the key. Already an issue for mobile devices where full disk encryption is widely used

#### Variants
- booting from live distributions  
Useful for some machines like laptops with non-standard interfaces or too difficult to disassemble properly. RAID arrays can also be accessed this way if they are managed by hardware controllers. we of course need to use a proper forensic distribution (e.g. Tsurugi, BackBox)
- system is powered on  
A system cannot always be turned off (critical systems) or we do not want to turn it off (live analysis of an intrusion). Keep in mind also that shutting down a machine may tamper with our evidence for example in the case of a fileless attacks. Another case is when disks are encrypted at rest so if we find them turned on we want to keep them on in order to access them. In this cases we first need to disconnect it from the network (cuts off a potential intruder) and save information in "volatility order":
  - dump memory
  - save runtime information: process info, network info
  - disk acquisition  
  NOTE: document all the steps since each command may alter the state
- live network analysis
Observe network traffic of a compromised machine to gather information about the author of the attack without accessing directly the machine itself in order not to make the attacker suspicious.

Further challenges:
- peculiarities of SSDs
- mobile device
- cloud forensics

### Identification
Tools:
- operating system
  - linux distributions
    - extensive native FS support
    - native support for hotswapping drives and mounting them
  - virtualization
    - set of VMs with windows version using SAMBA to share drives  

Rarely use windows directly because it tampers a lot with the drives and there is no native support for other FS. There are anyway tools for windows to perform certain operation (e.g. drive acquisition through encase). In some case it is not possibile to use SAMBA and we need to run the windows VM in non persistent mode.

To ensure repeatability and validation we should know how the tools we use work and that it is teoretically be reproduced by hands. It would be better to use open source software (not necessarily released with the source code but there needs to be the possibility to inspect its source code to check it)  

Common tasks in forensics analysis:
- retrieving deleted data: carving   
We can examine the bitstream image of the drive and look for sectors containing deleted data. (see how data is stored in a HDD). the carving technique consist of scanning the entire drive and look for file headers and footers of known file types. This way it is possible to retrieve files even when the original metadata to reach them have been deleted.  
Moreover since the OS allocates files in clusters which are composed by many disk sector there can be some "slack space" in the cluster that contains data from the previous file that was written on the cluster. This data maybe retrievable if it was a simple format like text, json or html. We can also try to match what we find against a specific file that we are looking for (if we find a 512 byte chunk that correspond bit by bit with another file that we are looking fore then we can be reasonably sure that the file was there)  
Tools for carving:
  - sleuth kit (autopsy for graphical interface) to analyze drive images, recover files, create timelines
  - gpart, testdisk: for partition recovery
  - photorec: to retrieve deleted photos

Remember that this can only give a positive confirmation  
The file is here  NOT ~~The file was not here before~~

#### Anti-forensics techniques
Aimed at creating confusion in the analyst, lead them off track or defeat the tools that he uses. There are two types of techniques:
- Transient (T) can be defeated if detected, doesn't destroy evidence -> interferes with identification
- Definitive (D) destroy evidence or make it impossible to acquire -> interferes with acquisition

Examples:  
TIMELINE TAMPERING (D)  
use tools (timestomp, touch) to modify creation date, access time and modification times of files, modifying the timeline of what has happened with the files

COUNTERING FILE RECOVERY (D) <-- most common  
- securely deleting files by overwriting them with 0
- wiping unallocated space
- use full disk encryption (FDE)
- using a VM in a cloud environment

FILELESS ATTACKS (D) <-- most common  
Do not leave traces of intrusions or attacks on the drive, inject malware in another process memory. All evidence is lost on shutdown, can only be noticed if we have a memory dump.

FILE SYSTEM INSERTION AND SUBVERSION TECHNOLOGIES (T)  
Hide data in places where it normally is not placed, for example inside file system metadata:
- partition tables
- using directory inodes (KY FS)
- adding ext3 journal on ext2 partition (WaffenFS)
- write in inodes marked as bad blocks (RuneFS)

LOG ANALYSIS (~T)  
Relies on the fact that logs are analyzed by automated tools. Insert something in the logs to make them fail to recognize patterns or try to exploit them

PARTITION TABLES TRICKS (T)  
- misaligned partition, may be missed by the analyst
- add multiple extended partition, sometimes tools fail to handle them
- generate many logical partition, same as above  
This happens because tools are often tested against reasonable partition schemes and may not handle extreme case.

Transient techniques in general relies on the fact that an analyst is not looking directly at a certain drive because of the size of data he has to analyze or time constraints. If an analyst is examining directly the drive it is very likely that they will be defeated.

#### Forensic charcteristcs of SSDs
SSDs are based on NAND memory that have particular characteristics
- limited lifespan (~10k writes)
- arranged in blocks that must be blanked before being rewritten

This lead manufacturers to optimize the duration and performance of the drive using a controller called FTL (flash translation layer):
- write caching
- trimming (blanking a block as soon as it is marked as deleted to speed up next writes)
- data compression (optimize lifespan)
- data encryption
- wear leveling (move data around the disk to wear all cells at the same rate)
- bad blocks handling (cells that have consumed all writing cycles)  
All of this is done directly by the FTL chip and it is transparent from the perspective of the OS. This means that the FTL can act independetly from the OS, as soon as it is powered on it can start to change and there is no way to bypass it by software.  
In theory we can read the NAND flash directly but it is extremely costly and time consuming and destroyes the drive. Even in this case, because of trimming, we cannot retrieve deleted data --> we lose carving.  
SSDs are also difficult to hash reliably because the FTL can sometimes reply with random data for unallocated blocks for performance optimization or because it is a cheap FTL.

### Evaluation
In this phase we need to match evidence elements (facts) wiht the required elements to support or negate a legal theoay. Cooperation is required between the expert and lawyer.  
What to evaluate:
- elements that support the indictment
- possible alternative explaination
- analyze what can be said, what can't be said and what further experiments would be needed to say more  
  - This last point is particularly important because we should assess toghether with the lawyer what we want to analyze and what risks does this involve for our client. For example if i'm defending someone accused of illegally accessing a system. If his laptop has been seized and not yet analyzed we need to discuss with the lawyer if we want to analyze it or not that the opponent may have it analyzed (remember that it is an adversarial setting)

#### Relationship with the different entities
RELATIONSHIP WITH LAWYER:
- lawyers own the choice of the defense strategy
  - they may ask for suggestion
- lawyers own relationship with the client
  - never tarnish the trust relation with the client
- lawyers do not dictate what to write
  - must NOT lie -> expert witness can commit perjury if he does
  - maybe asked to omit things as long as it is not the same as lying

RELATIONSHIP WITH CUSTOMER:
- assist the part of the judgement that hired you
  - find what helps the customer -> this is how law should work
  - this helps the judge in making the correct decision
- process truth != historical truth
  - it is better to let a potential criminal away than to have an innocent person jailed

RELATIONSHIP WITH PROSECUTOR:
- assisting prosecutor does not entail moral superiority
- remember to stick to facts, do not get your words shaped by justice

#### Analyzing previous documents
Review documents that have already been presented in the proceedings, in particular reports of other experts.
What to look for:
- technical/factual errors
  - acquisition
    - search and seizure process
    - serial numbers
    - hashing/cloning procedures (use of write blockers, missing hashes)
  - identification
    - integrity not verified
    - proprietary tools
    - technical mistakes
    - known bugs in tools
- unclear reasoning or methodologies
- suggestive writing
  - did not explore possibile alternative explainations of the facts
  - look for counter examples for the assumptions
  - are there missing explainations?
- not clear distinction between facts and theory/hypotheses

### Presentation
Writing the report is extremely important because it will be analyzed by the court and the other party's expert.  
- be clear
- don't overdo with english terminology and explain technical terms in footnotes
- be simple without being simplistic (don't make people feel stupid)
- explain why what you are writing is relavant, preferrably before starting the discussion of the experiment (give motivation to make the efforto to read and understand)
- remeber to explore possible alternatives and defend against possible counter-measures (defensive writing)  
What not to do:
- be overly technical, you will be ignored
- use innuendos or sarcasm
- use weak arguments when you have a better ones
- show bias with one of the two parties, be emotionless in the writing (especially do not empatize with the victims in the report)
- say things and not explaining them
- be excessively deferent to the judge

Structure of the report  
Model it on a scientific report but also structure as an "obstacle course" with each obstacle being a little higher than the previous to make it difficult for the judge to ignore your arguments. Example:
1. Foreword: state what you have examined and what you was asked to report on.
2. Introduction: brief summary of what the report is going to be on and why (a sort of abstract).
3. Acquisition issues and what could have happened because of this.
4. On the technical analysis, give alternate explainations for the facts, missing evidence, experiments that could have been done that could chenge the understanding of the facts.
5. Conclusions, this is likely the only part that the judge will read, recap what was shown in the report and state what can be concluded in a single strong phase.

### Testimony as a expert witness
In many jurisdiction expert witnesses provides a sworn testimony and can commit perjury and cannot "hide" behind professional secrecy. The examination usually has two phases:
- direct examination  
Called by your side, is the "friendly" part of the examination. Often it is prepared in advance with the lawyer. The approach during this phase is to take your time and explain everithing to the judge being very clear and helpful. In Italy it is also possible for the judge to ask questions of their own, be prepared.

- cross examination  
Unfriendly or outright hostile. Refer to your report to answer questions. If possibile reply with yes/no, otherwise be very complex and difficult to understand. Never get angry, even if competency is called into question (it is basically standard procedure).

Remember to never hide things that you know if you are asked, perjury is a crime!
