PLANETARY DAO
Technical White Paper 
2022-06-15
 Joanna A. Picetti§  joanna@jp.network


W3ID - Planet Earth Citizenship v0.1 
A Proof-of-Unique-Human System
                                                    
                                                     “The Rights of Citizenship”
Abstract. Planet Earth Citizenship is based on pseudonym events, global and simultaneous verification events that occur at the exact same time for every person on Earth. In these events, people are randomly paired together, 1-on-1, to verify that the other is a person, in a pseudo-anonymous context. 


Introduction 

Planetary Citizenship is a population registry for a new global society that is built on top of the internet. It provides a simple way to give every human on Earth a proof-of-unique-human, and does so in a way that cannot exclude or reject a human being, as long as the average person in the population would recognize them as human. This means it is incapable of shutting anyone out, and that it will inherently form a single global population record, that integrates every single human on Earth. It is incapable of discrimination, because it is incapable of distinguishing one person from another, since it has absolutely zero data about anyone. It’s incapable of discriminating people by gender, sexuality, race or belief. The population registry also has infinite scalability. The pairs, dyads, are autonomous units and the control structure of Planetary Citizenship. Each pair is concerned only with itself, compartmentalized, operating identically regardless of how many times the population doubles in size. 
Pseudonym events, a new population registry

With global and simultaneous pseudonym events, it is possible to prove that every person on Earth only has one proof-of-unique-person within the system. The ideal organization is that people form pairs. Each person is paired with a stranger, using a new public key that is untraceable to the account.  During the event, both partners in a pair have to mutually verify one another, using the verify() function. If a person has any reason to not verify the account they are paired with, they can use the dispute() function to break up their pair, and each be assigned to a “court” under another pair.



The “court” system, and the “virtual border”

The key to Planetary Citizenship is the “court” system, that subordinates people under a random pair, to be verified in a 2-on-1 way. This is used in two scenarios. The first, is when a person is paired with an account that does not follow protocol. In most cases, this would be a computer script, or a person attempting to participate in two or more pairs at the same time. Normally, in the pairs, people have to mutually verify each other to be verified. In case a person is paired with an attacker, they can choose to break up their pair, and subordinate both people in the pair under a random pair each, a “court”. The attacker, if they were a bot, will not be verified by the court, or have any ability to coerce it, while the normal person will be verified by their “court”. Both people in the pair that “judges” a court have to verify the person being judged.

struct Court { uint id; bool[2] verified; }

mapping (uint => mapping (address => Court)) public court;

function dispute() external {
uint t = schedule()-1;
uint id = getPair(nym[t][msg.sender].id);
require(id != 0);
require(!pairVerified(t, id));
pair[t][id].disputed = true;
}

The second scenario for the “court” system is the “virtual border”. The population has a form of “border” or “wall” around it, and anyone not verified in the previous event is on the “outside” of this “border”. To register, you need to go through an "immigration process". During this process, you are assigned to a “court”, another pair, and they verify you in a 2-on-1 way, so that a bot would have no way to intimidate or pressure this “border police”. This “border”, together with the dispute mechanism, acts as a filter that prevents any attackers to the system.


2, 4, 8, 16… 1 billion, growth by doubling

The pairs are the control structure of Planetary Citizenship, and each pair is concerned only with itself, regardless of how many times the population doubles in size. The population is allowed to double in size each event, made possible by that each person verified during an event is authorized to invite another person to the next event. This allows the population to grow from 2 to 10^3 in 10 events, 10^3 to 10^6 in 20 events, and 10^6 to 10^9 in 30 events.
There is no theoretical upper limit to this scalability mechanism, Planetary Citizenship has infinite scalability. The constructor allows the contract to be initialized from two people, and can also be used when transferring the population from one-ledger to another as digital ledgers get more advanced.

constructor(uint _population, address _genesis) {

   balanceOf[0][Token.Registration][_genesis] = _population;
}


Randomization, Vires in Numeris

The population is randomized by that each person who invokes register() will take the place of a randomly selected person, and move that person to the end. This mechanism keeps the computational cost per person low. The registration is open during the entire period, and the shuffle is finished at the end of the period when the registration closes. Once shuffle is complete, the first two people in the registry list will form pair 1, the next two pair 2, the next two pair 3, etc. The pair of any account can be calculated with the getPair() function.


uint entropy;

function getRandomNumber() internal returns (uint) {
return entropy = unit(keccak256(abi.encode(blockhash(block.number - 1), entropy)));
}

struct Nym { uint id; bool verified; }

mapping (uint => mapping (address => Nym)) public nym;
mapping (uint => address[]) public registry;

function register() public {
uint t = schedule();
deductToken(t, Token.Registration);
registry[t].push();
uint counter = registered(t);
uint id = 1 + getRandomNumber()%counter;
registry[t][counter-1] = registry[t][id-1];
registry[t][id-1] = msg.sender;
nym[t][registry[t][counter-1]].id = counter;
nym[t][msg.sender].id = id;
}



Collusion attacks

Planetary Citizenship is vulnerable to collusion attacks. The success of collusion attacks increases quadratically, as x², where x is the percentage colluding. Repeated attacks conform to the recursive sequence a[n] == (x + a[n — 1])²/(1 + a[n — 1]), and can be seen to approach n as x -> 1. It plateaus at the limit a[∞] = x²/(1−2x) for 0<x<0.5. Colluders reach 50% control when (a[∞] + x) == (1+a[∞])/2, this happens at x = 1/3, i.e., Planetary System is a 66% majority controlled system.



An anonymous population registry

Since “who a person is” is not a factor in the proof, mixing of the proof-of-unique-human does not reduce the reliability of the protocol in any way. It is therefore allowed, and encouraged. This is practically achieved with “tokens” that are intermediary between verification in one pseudonym event and registration for the next event, authorizing mixer contracts to handle your “token” using the approve() function.


enum Token { Personhood, Registration, Immigration }

mapping (uint => mapping (Token => mapping (address => uint))) public balanceOf;
mapping (uint => mapping (Token => mapping (address => mapping (address => uint)))) public allowed;

function _transfer(uint _t, address _from, address _to, uint _value, Token _token) internal {
require(balanceOf[_t][_token][_from] >= _value);
balanceOf[_t][_token][_from] -= _value;
balanceOf[_t][_token][_to] += _value;
}
function transfer(address _to, uint _value, Token _token) external {
_transfer(schedule(), msg.sender, _to, _value, _token);
}
function approve(address _spender, uint _value, Token _token) external {
allowed[schedule()][_token][msg.sender][_spender] = _value;
}
function transferFrom(address _from, address _to, uint _value, Token _token) external {
uint t = schedule();
require(allowed[t][_token][_from][msg.sender] >= _value);
_transfer(t, _from, _to, _value, _token);
allowed[t][_token][_from][msg.sender] -= _value;
}


Proof-of-unique-human as a commodity

The proof-of-unique-human is only valid for one month, untraceable from month to month, and disposable once expired. The population registry has no concept of “who you are”. It cannot distinguish one person from another.Any other contract can build applications based on this proof-of-unique-human just by referencing proofOfUniqueHuman[_period][_account]. Applications that use some kind of majority vote, can reference population[_period] to know how many votes are needed to be >50% of the population.

mapping (uint => uint) public population;
mapping (uint => mapping (address => bool)) public proofOfUniqueHuman;

function claimPersonhood() external {
uint t = schedule();
deductToken(t, Token.Personhood);
proofOfUniqueHuman[t][msg.sender] = true;
population[t]++;
}


Man-in-the-middle attacks, and “pre-meetings”

Man in the middle attacks are when two fake accounts relay the communication between the two real humans the fake accounts are chosen to verify. Then the two real humans each verified a bot. These are defended against simply by the real people asking each other what pair they are in. But video manipulation attacks have to be considered. To have extra security, a “handshake” to secure the channel is added, in a way that is as “Turing safe” (same difficulty for breaking Turing test) as the actual event itself. 


Public keys are exchanged along with the numbers, as part of the encrypted message. Before exchanging encryption keys, the pair also meets (otherwise, the mechanism can be attacked. ) The pair then decrypts their numbers, and meet at the time that the random number specifies. This “pre-meeting” can only take place if they got the same number, and it proves that an honest message could be exchanged. The public keys that were included in the message are used to secure the channel.




function halftime(uint _t) public pure returns (bool) { return(block.timestamp > toSeconds(_t)+period/2); }

















References

Pseudonym Parties: An Offline Foundation for Online Accountable Pseudonyms,
https://pdos.csail.mit.edu/papers/accountable-pseudonyms-socialnets08.pdf (2008)
Pseudonym Pairs: A foundation for proof-of-personhood in the web 3.0 jurisdiction,
https://panarchy.app/PseudonymPairs.pdf (2018)





