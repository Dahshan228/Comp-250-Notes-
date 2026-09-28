---
title: The Limits of Certificate Revocation in Vehicle to Vehicle Communication
author: Ali Eldahshan
date: September 11, 2026
institution: Department of Computer Science, Loyola University Chicago
course: "COMP-348: Network Security"
instructor: Dr. Corby Schmitz
bibliography: "Essay Revision- The Limits of Certificate Revocation in Vehicle to Vehicle Communication.json"
csl: apa.csl
abstract: |
  Vehicle-to-vehicle (V2V) communication warns drivers of hazards they cannot see,
  and the Security Credential Management System (SCMS) uses certificates to
  ensure that each warning comes from a legitimate vehicle.
  But a certificate authenticates the sender, not the truth of the message, and
  a legitimate vehicle can still broadcast false data.
  This paper argues that SCMS's response, revoking the misbehaving vehicle's
  certificates, cannot prevent the first false message, because revocation begins
  only after reports accumulate and the list reaches other vehicles.
  The remaining defense lies with receiving vehicles, which must check messages
  against their own sensors, though a careful attacker can evade even those checks.
---

A driver approaches an intersection where buildings block the view to the left and right.
They cannot see if a car is coming from either direction.
A warning from their vehicle tells them to brake because a car is approaching from the right and is not slowing down.
The warning comes from vehicle-to-vehicle (V2V) communication, in which cars broadcast their position and speed, and each message carries a certificate from the Security Credential Management System (SCMS) proving it came from a legitimate vehicle.
They cannot check whether the warning is accurate, because their vision is blocked.
This matters because if the other car continued on and they did not brake, they would be hit.<!--Context of my argument-->
If the warning is false, the driver brakes hard for a hazard that was never there, and the car behind may not stop in time.
The driver cannot tell the two situations apart, which leaves the certificate as the only thing separating a real warning from a fabricated one.<!--Problem addressed in my argument-->
Certificates authenticate the sender of a message but don't say anything about whether its contents are true, and SCMS\'s answer to a misbehaving vehicle, revoking its certificates and distributing them on a revocation list, cannot prevent the first false message it sends.<!--My claim-->

# How V2V Messages Are Trusted

A basic safety message (BSM) is broadcast by a vehicle up to ten times per second and contains the sender\'s time, position, speed, and path history [@brechtSecurityCredentialManagement2018].
Each message is digitally signed, and a receiving vehicle verifies that signature before acting on the message, which establishes that the sender is a real and verified participant in the network.
The signature does not establish whether the contents of the message are true.

V2V communication is a connected vehicle technology rather than an autonomous one.
The vehicles exchanging messages are driven by people, and the system\'s output is a warning that the driver receives and decides how to act on.
This technology is most useful when a driver has no line of sight to the hazard.
If a vehicle several cars ahead brakes hard on a highway, drivers further back cannot see it happen and may not react in time.
A broadcast message reaches them immediately, because radio signals are not blocked by the vehicles in between.

When a vehicle receives a BSM, it uses the data to determine whether a hazard exists, and if it does, it generates a warning for the driver.
This is what makes false data dangerous as the receiving vehicle acts on the contents of the message, not just the validity of its signature.

V2X is a general term for vehicle communications, which includes vehicle to vehicle, vehicle to infrastructure, vehicle to pedestrian, and vehicle to network communication [@gyawaliMisbehaviorDetectionUsing2019].
These systems support driver assistance applications such as forward collision warning and curve speed warning.
This paper focuses on V2V communication rather than V2X as a whole.

# Certificates and the SCMS

A vehicle first sends a request to join the V2X network, and SCMS approves it and issues an enrollment certificate [@autocryptWhatSecurityCredential2021].
The vehicle keeps the enrollment certificate onboard as proof that it is authorized, and uses it to request the pseudonym certificates it will attach to its messages.
A pseudonym certificate lets a receiver verify the signature without revealing which vehicle sent the message.
@gyawaliMisbehaviorDetectionUsing2019 describe the same registration process, in which a vehicle obtains a unique identity, pseudonym identities, a private and public key pair, and a certificate issued by the certificate authority.

If a vehicle used one permanent certificate, anyone operating receivers along a road could recognize the same certificate at multiple points and figure out the vehicle\'s movements over time, including where its driver lives and works.

When a message arrives, the receiving vehicle verifies the attached certificate and checks it against the list of revoked certificates, and accepts the message only if both checks pass [@autocryptWhatSecurityCredential2021].

AUTOCRYPT describes the pseudonym certificate as encrypted, but this cannot be accurate, since a receiving vehicle must be able to read the certificate in order to verify it.
What the pseudonym certificates provide is anonymity rather than encryption as the certificate is readable by any receiver but does not identify the vehicle that holds it.
This oversight might be due to the fact that AUTOCRYPT sells V2X security products, so its description of SCMS is promotional material rather than independent analysis.

# How Revocation Works

The problem is that the pseudonym certificates are deliberately unlinkable, so identifying a misbehaving vehicle from one reported certificate is not enough to revoke the rest of the batch it holds.
@brechtSecurityCredentialManagement2018 identify efficient revocation as \"one of the main challenges\" given the number of pseudonym certificates each vehicle holds.
This challenge is not coincidental, because the same unlinkability that protects drivers' privacy also leaves revocation in the hands of the Misbehavior Authority.

A vehicle with valid certificates broadcasts a message containing false information.
Because V2X relies on vehicles accepting data from any sender holding a valid certificate, the system is vulnerable to attacks originating from users inside it.
@gyawaliMisbehaviorDetectionUsing2019 note that cryptographic methods have been found less effective against these insider attacks.
When a vehicle receives a message whose contents are not possible, it reports the sending certificate to the Misbehavior Authority.
The Misbehavior Authority does not act on a single report, since one anomaly could be a failing sensor rather than an attack, so it waits until enough reports add up to justify revocation.
The Misbehavior Authority uses the linkage mechanism to trace the reported pseudonym certificate back to the vehicle and identify every other pseudonym certificate it holds, then adds them all to the certificate revocation list.
The revocation list is then distributed to vehicles, and each one must receive and load it before it will begin rejecting messages signed with those certificates.

# Why the First Message Gets Through

In a false alert generation attack, a malicious vehicle broadcasts a fabricated warning to its neighbors, such as an emergency brake light or a collision warning, which may disrupt traffic or cause an accident [@gyawaliMisbehaviorDetectionUsing2019].
In a position falsification attack, the attacker alters the position information in its own broadcast beacons.
@gyawaliMisbehaviorDetectionUsing2019 claim that this can be used to create multiple identities at fabricated locations, which may cause traffic accidents.
This paper focuses on false alert attacks, but the timing problem applies equally to both.
In either case, a vehicle can only be added to a blacklist after it has already transmitted a false message, so the first message always reaches its targets.

Better engineering can speed up every step of revocation except the review.
That step is slow by design, because the Misbehavior Authority must wait for enough reports to accumulate before it acts.
If it revoked a vehicle on the first report, it could remove that vehicle from the network over a single irregular reading rather than a real pattern of misbehavior.
The wait protects honest vehicles from wrongful revocation, but while the Authority waits, the misbehaving vehicle continues to broadcast false messages.

Unlike the centralized revocation list used by SCMS, the misbehavior detection system described by @gyawaliMisbehaviorDetectionUsing2019 has a vehicle that detects an attacker broadcast a warning to nearby vehicles, which then add the accused vehicle to an accusation list.
Once enough warnings about a vehicle add up, its ID is added to a local blacklist and nearby vehicles are instructed to ignore its broadcasts.
Permanent exclusion requires the local blacklist to reach the certificate authority, and that can only happen when a vehicle carrying the local blacklist comes in range of a roadside unit or base station [@gyawaliMisbehaviorDetectionUsing2019].
This delay cannot be reduced by better engineering, because it depends on where a driver happens to drive rather than on the speed of the system.

# Conclusion

The driver at the intersection would still receive the false message, because the entire process only begins after that message has been broadcast.
Revocation does prevent a misbehaving vehicle from continuing to send false messages.
AUTOCRYPT describes SCMS as maintaining a record of revoked devices that \"helps prevent the same threats from reoccurring\" [@autocryptWhatSecurityCredential2021].
Stopping a vehicle from continuing to broadcast does nothing for the driver who has already acted on the first false message.

Certificates prevent outsiders from injecting messages into the network, but SCMS\'s response to a legitimate vehicle that begins broadcasting false data is entirely reactive, and the first false message reaches drivers before any of it takes effect.
If SCMS cannot prevent the first false message, the remaining defense lies with the receiving vehicle, which can check incoming messages against its own sensor data and reject those that are not possible.
@gyawaliMisbehaviorDetectionUsing2019 complicate this defense by describing an attacker who transmits low speed and traffic flow values that are consistent with current traffic conditions, allowing a false alert to pass the receiving vehicle\'s own possibility checks.
Until receiving vehicles can judge whether a signed message is plausible and not just whether it is authentic, the first false warning will reach the driver, and the question V2V security must answer is how a car can doubt a message it has every cryptographic reason to trust.

# Bibliography
