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
