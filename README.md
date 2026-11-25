[README_Proverif.md](https://github.com/user-attachments/files/35396030/README_Proverif.md)
**CBDC Payment Protocol: Formal Verification Starter Models**



OVERVIEW



This repository contains a small starter model of a central bank digital
currency (CBDC) payment, written in the protocol verification tool
ProVerif. The goal is to learn what ProVerif can and cannot express, and
to build a base for richer models later.



THE PROTOCOL



There are three roles.

1\. Alice creates a fresh transaction id, signs a payment request

&#x20;  (transaction id, Alice, Bob, amount) and sends it.

2\. The issuer checks Alice's signature. If the transaction id has not

&#x20;  been spent before, it records the id in a ledger, accepts the payment

&#x20;  and sends a confirmation.

3\. Bob receives the confirmation.



The attacker controls the network (Dolev-Yao model). Alice's signing key

is secret. Everything else is public. The confirmation sent to Bob is

unsigned in this version, on purpose, to see what the tool report.



FILES



proverif/cbdc\_nolock.pv

&#x20; ProVerif model. The ledger is a table, with a check (get) followed by

&#x20; an insert. This is the main ProVerif model.



proverif/cbdc\_lock.pv

&#x20; Same as above, plus a private channel used as a lock around the ledger

&#x20; check. This is an experiment, kept to show a limit of ProVerif.



results/

&#x20; Attack traces and a written summary of the findings.



PROPERTIES CHECKED



payment\_integrity: the issuer only accepts payments that Alice initiated.

no\_replay: the same payment is never accepted twice.

executable / reachability: the protocol can run.

confirmation authenticity (ProVerif only): Bob only receives

&#x20; confirmations that the issuer really produced.



RESULTS



ProVerif (cbdc\_nolock.pv):

&#x20; payment\_integrity            true

&#x20; no\_replay (injective query)  false: replay trace found

&#x20; Received reachable           yes (reached by a forged confirmation)

&#x20; Received implies Accepted    false: forged confirmation trace found



ProVerif (cbdc\_lock.pv):

&#x20; payment\_integrity            true

&#x20; no\_replay (injective query)  cannot be proved

&#x20; the other two queries        cannot be proved



KEY FINDINGS



1\. Replay protection depends on an atomic ledger update. In ProVerif,

&#x20;  the check of the table and the insert are two separate steps, so two

&#x20;  issuer instances can both see an empty ledger and both accept the same

&#x20;  signed payment. Signatures are not broken. The weakness is the race.



2\. The unsigned confirmation can be forged. Anyone can send Bob a message

&#x20;  of the form (id, PaymentAccepted, Bob, ten) and Bob accepts it.



3\. ProVerif cannot express a lock. Messages are treated as reusable, so a

&#x20;  single lock token can be taken by many sessions. ProVerif then reports

&#x20;  "cannot be proved" and finds no trace. "Spent exactly once" is a linear

&#x20;  resource property.



4\. The ProVerif reachability query only shows that the Received event can

&#x20;  happen, and the trace goes through a forged message. It does not show

&#x20;  the honest run from Alice to the issuer to Bob.



HOW TO RUN



ProVerif:

&#x20; proverif proverif/cbdc\_nolock.pv

&#x20; proverif proverif/cbdc\_lock.pv



ProVerif with HTML output and attack graphs (create the folder first):

&#x20; mkdir -p out

&#x20; proverif -html out proverif/cbdc\_nolock.pv

&#x20; Then open out/index.html in a browser.

&#x20; Graphs need Graphviz (sudo apt install graphviz).



LIMITATIONS



This is a starter model, not a model of a real CBDC. The amount is a

fixed constant, there is one payer and one payee, there is no privacy

property yet, and the issuer is trusted. The findings are about this

model only.



PLANNED WORK



v3: the issuer signs the confirmation and Bob verifies it, so that the

&#x20;   forgery finding disappears.

v3: an honest-run reachability query (Accepted before Received).

Next: an anonymity and unlinkability model in ProVerif using choice\[...].

Later: zero-knowledge proofs and threshold signatures.



TOOLS



ProVerif, version: Proverif 2.05. Cryptographic protocol verifier, by Bruno Blanchet, Vincent Cheval, and Marc Sylvestre



👩‍💻 Author 


Maham Nasir MS Cybersecurity, COMSATS University Islamabad, Pakistan Researcher in Post-Quantum Cryptography & Privacy-Preserving Protocols

LinkedIn: https://www.linkedin.com/maham-n-0315

"This project is part of ongoing independent research into post-quantum cryptographic protocol design, with applications to privacy-preserving digital currency systems (CBDCs) and secure identity protocols using formal verification."

