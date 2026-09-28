**The Limits of Certificate Revocation in Vehicle to Vehicle Communication**

Ali Eldahshan

Department of Computer Science, Loyola University Chicago

COMP-348: Network Security

Dr. Corby Schmitz

September 11, 2026

A driver can see an open road that has buildings blocking their view of the left and right of the intersection. They cannot see if a car is coming from either direction. A warning from their vehicle tells them to brake because a car is approaching from the right and is not slowing down. They cannot check whether the warning is accurate, because their vision is blocked. This matters because if the other car continued on and they did not brake, they would be hit. If the warning is false, the driver brakes hard for a hazard that was never there, and the car behind may not stop in time. The driver cannot tell the two situations apart, which leaves the certificate as the only thing separating a real warning from a fabricated one. [Certificates authenticate the sender of a message but don't say anything about whether its contents are true, and SCMS\'s answer to a misbehaving vehicle, revoking its certificates and distributing them on a revocation list, cannot prevent the first false message it sends.]{.mark}

A basic safety message (BSM) is broadcast by a vehicle up to ten times per second and contains the sender\'s time, position, speed, and path history (Brecht et al., 2018). Each message is digitally signed, and a receiving vehicle verifies that signature before acting on the message, which establishes that the sender is a real and verified participant in the network. The signature does not establish whether the contents of the message are true.

V2V communication is a connected vehicle technology rather than an autonomous one. The vehicles exchanging messages are driven by people, and the system\'s output is a warning that the driver receives and decides how to act on. This technology is most useful when a driver has no line of sight to the hazard. If a vehicle several cars ahead brakes hard on a highway, drivers further back cannot see it happen and may not react in time. A broadcast message reaches them immediately, because radio signals are not blocked by the vehicles in between.

When a vehicle receives a BSM, it uses the data to determine whether a hazard exists, and if it does, it generates a warning for the driver. This is what makes false data dangerous as the receiving vehicle acts on the contents of the message, not just the validity of its signature.

V2X is a general term for vehicle communications, which includes vehicle to vehicle, vehicle to infrastructure, vehicle to pedestrian, and vehicle to network communication (Gyawali & Qian, 2019). These systems support driver assistance applications such as forward collision warning and curve speed warning. This paper focuses on V2V communication rather than V2X as a whole.

A vehicle first sends a request to join the V2X network, and SCMS approves it and issues an enrollment certificate (AUTOCRYPT, n.d.). The vehicle keeps the enrollment certificate onboard as proof that it is authorized, and uses it to request the pseudonym certificates it will attach to its messages. A pseudonym certificate lets a receiver verify the signature without revealing which vehicle sent the message. Gyawali and Qian (2019) describe the same registration process, in which a vehicle obtains a unique identity, pseudonym identities, a private and public key pair, and a certificate issued by the certificate authority.

If a vehicle used one permanent certificate, anyone operating receivers along a road could recognize the same certificate at multiple points and figure out the vehicle\'s movements over time, including where its driver lives and works.

When a message arrives, the receiving vehicle verifies the attached certificate and checks it against the list of revoked certificates, and accepts the message only if both checks pass (AUTOCRYPT, n.d.).

AUTOCRYPT describes the pseudonym certificate as encrypted, but this cannot be accurate, since a receiving vehicle must be able to read the certificate in order to verify it. What the pseudonym certificates provides is anonymity rather than encryption as the certificate is readable by any receiver but does not identify the vehicle that holds it. This oversight might be due to the fact that AUTOCRYPT sells V2X security products, so its description of SCMS is promotional material rather than independent analysis.

The problem is that the pseudonym certificates are deliberately unlinkable, so identifying a misbehaving vehicle from one reported certificate is not enough to revoke the rest of the batch it holds. Brecht et al. (2018) identify efficient revocation as \"one of the main challenges\" given the number of pseudonym certificates each vehicle holds.

A vehicle with valid certificates broadcasts a message containing false information. Because V2X relies on vehicles accepting data from any sender holding a valid certificate, the system is vulnerable to attacks originating from users inside it. Gyawali and Qian (2019) note that cryptographic methods have been found less effective against these insider attacks. When a vehicle receives a message whose contents are not possible, it reports the sending certificate to the Misbehavior Authority. The Misbehavior Authority does not act on a single report, since one anomaly could be a failing sensor rather than an attack, so it waits until enough reports add up to justify revocation. The Misbehavior Authority uses the linkage mechanism to trace the reported pseudonym certificate back to the vehicle and identify every other pseudonym certificate it holds, then adds them all to the certificate revocation list. The revocation list is then distributed to vehicles, and each one must receive and load it before it will begin rejecting messages signed with those certificates.

In a false alert generation attack, a malicious vehicle broadcasts a fabricated warning to its neighbors, such as an emergency brake light or a collision warning, which may disrupt traffic or cause an accident (Gyawali & Qian, 2019). In a position falsification attack, the attacker alters the position information in its own broadcast beacons. Gyawali and Qian (2019) claim that this can be used to create multiple identities at fabricated locations, which may cause traffic accidents. This paper focuses on false alert attacks, but the timing problem applies equally to both. In either case, a vehicle can only be added to a blacklist after it has already transmitted a false message, so the first message always reaches its targets.

Every step in this process can be sped up with better engineering except the review step, because the delay there is not a technical limitation but a deliberate one the Misbehavior Authority have to wait for enough reports to accumulate before it can act. If the authority revoked on the first report, it could remove a vehicle from the network over a single irregular reading rather than a real pattern of misbehavior. The wait is needed to avoid wrongfully revoking a vehicle, however, during that wait the misbehaving vehicle continues to broadcast false messages.

Unlike the centralized revocation list used by SCMS, the misbehavior detection system described by Gyawali and Qian (2019) has a vehicle that detects an attacker broadcast a warning to nearby vehicles, which then add the accused vehicle to an accusation list. Once enough warnings about a vehicle add up, its ID is added to a local blacklist and nearby vehicles are instructed to ignore its broadcasts. Permanent exclusion requires the local blacklist to reach the certificate authority, and that can only happen when a vehicle carrying the local blacklist comes in range of a roadside unit or base station (Gyawali & Qian, 2019). This delay cannot be reduced by better engineering, because it depends on where a driver happens to drive rather than on the speed of the system.

The driver at the intersection would still receive the false message, because the entire process only begins after that message has been broadcast. Revocation does prevent a misbehaving vehicle from continuing to send false messages. AUTOCRYPT describes SCMS as maintaining a record of revoked devices that \"helps prevent the same threats from reoccurring\" (AUTOCRYPT, n.d.). Stopping a vehicle from continuing to broadcast does nothing for the driver who has already acted on the first false message.

Certificates prevent outsiders from injecting messages into the network, but SCMS\'s response to a legitimate vehicle that begins broadcasting false data is entirely reactive, and the first false message reaches drivers before any of it takes effect. Revocation is effective at stopping a misbehaving vehicle from sending further harmful messages. If SCMS cannot prevent the first false message, the remaining defense lies with the receiving vehicle, which can check incoming messages against its own sensor data and reject those that are not possible. Gyawali and Qian (2019) complicate this defense by describing an attacker who transmits low speed and traffic flow values that are consistent with current traffic conditions, allowing a false alert to pass the receiving vehicle\'s own possibility checks.

**References**

**AUTOCRYPT. (n.d.). *What is the security credential management system?* <https://www.autocrypt.io/security-credential-management-system/>**

**Brecht, B., Therriault, D., Weimerskirch, A., Whyte, W., Kumar, V., Hehn, T., & Goudy, R. (2018). A security credential management system for V2X communications. *IEEE Transactions on Intelligent Transportation Systems*, *19*(12), 3850-3871. [1802.05323](https://arxiv.org/pdf/1802.05323)**

**Gyawali, S., & Qian, Y. (2019). Misbehavior detection using machine learning in vehicular communication networks. *ICC 2019 - 2019 IEEE International Conference on Communications*, 1-6. <https://doi.org/10.1109/ICC.2019.8761300>**
