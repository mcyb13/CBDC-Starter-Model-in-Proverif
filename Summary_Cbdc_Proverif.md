**Summary of Findings**



SCOPE



This summary covers three ProVerif attack traces and the lock experiment

for the CBDC starter model.



TRACE 1: REPLAY OF ONE PAYMENT (no\_replay is false)



Query: every acceptance of a payment must match a different initiation by

Alice (injective correspondence).



What happens:

&#x20; Alice creates one fresh transaction id and sends one signed payment.

&#x20; The attacker delivers the same message to two issuer instances.

&#x20; Both check the spent table while it is empty and take the else branch.

&#x20; Both insert the id and both fire the Accepted event.



Result: one Initiated event and two Accepted events.



Meaning: the signature is not broken and nothing is forged. The check

and the insert are separate steps, so the instances interleave. The

weaker property (every accepted payment was initiated by Alice) still

holds. Replay protection needs an atomic ledger update.



TRACE 2: BOB'S CONFIRMATION IS REACHABLE (Received is reachable)



Query: can the Received event happen at all?



What happens:

&#x20; Only the setup runs. The attacker builds a message of the form

&#x20; (any value, PaymentAccepted, Bob, ten) and sends it to Bob.

&#x20; Bob fires the Received event.



Result: reachable.



Meaning: this shows the event can occur, but through a forged message.

It is weaker than the Tamarin executable lemma, which needs Accepted

before Received. An honest-run query should be added in version 3.



TRACE 3: FORGED CONFIRMATION (Received implies Accepted is false)



Query: whenever Bob receives a confirmation, the issuer accepted that

payment earlier.



What happens: the same attack as trace 2.



Result: Received happens with no Accepted.



Meaning: the confirmation is unsigned, so anyone can create it. The fix

is for the issuer to sign the confirmation and for Bob to verify the

signature against the issuer's public key.



WHY TRACES 2 AND 3 LOOK THE SAME



Both queries are broken by the same shortest attack: one forged message

sent to Bob.



LOCK EXPERIMENT



In cbdc\_lock.pv, a private channel carries a single token that an issuer

instance must take before checking the ledger and release afterwards.

ProVerif answers "cannot be proved" for the replay query and produces no

trace, so there is no graph for this model.



Reason: ProVerif treats a message as reusable, so the single token can be

taken by several sessions at once. It cannot count or consume resources.

When it tries to rebuild a real execution from its derivation, it fails,

because in a real run the token would be consumed. The result is

"cannot be proved" and not "false".



Conclusion: "spent exactly once" is a linear-state property. Tamarin can

express it with facts that are consumed. ProVerif cannot, directly.



TABLE OF FINDINGS



Trace 1

&#x20; Query:    no\_replay (injective)

&#x20; Finding:  race between ledger check and insert

&#x20; Fix:      atomic ledger update (assumed in Tamarin)



Trace 2

&#x20; Query:    reachability of Received

&#x20; Finding:  reachable, but only through a forged message

&#x20; Fix:      add reachability of Accepted for the honest run



Trace 3

&#x20; Query:    Received implies Accepted

&#x20; Finding:  unsigned confirmation can be forged

&#x20; Fix:      issuer signs the confirmation (version 3)



Traces file name in PDF

Trace 1: trace1\_cbdc\_nolock\_proverif

Trace 2: trace2\_cbdc\_nolock\_proverif

Trace 3: trace3\_cbdc\_nolock\_proverif

