# Frequency Capping

## Summary

- This proposal puts forward a solution to frequency capping, which doesn't just adhere to the privacy-by-design principles of the web browsers, but develops them
- Frequency capping will be partitioned by domain, unless browsers present an alternative which will by nature be identity-less (e.g. Google's [FLoCs](https://github.com/jkarlin/floc) and [TURTLEDOVE](https://github.com/michaelkleber/turtledove) proposals)
- Using a Publisher's ID stored in a first-party cookie provides a short-term fix for frequency capping, but fails to protect user privacy to the standards laid out by the browsers (e.g. Google's Privacy Sandbox) and leaks Publisher's first-party data and the value it creates to the rest of the ecosystem.
- By moving the execution of frequency capping logic from the DSP to the Publisher's header bidding wrapper or SSP or ad server, ad-tech can
    1. implement per-domain frequency capping
    2. remove the need for identity, meeting the privacy-by-design principles laid out by the browsers and
    3. prepare itself for a potential future-state where browsers allow for cross-domain frequency controls.

## Introduction

Third-party cookies support three general use cases:

- Targeting
- Frequency capping
- Measurement and attribution

As third-party cookies disappear, finding tangible solutions rooted in privacy becomes critical. Any proposal will (1) require support from the browsers by furthering their policies and (2) must work towards the principles of regulations like the GDPR. Both of these requirements lead to a third requirement: that any solution must be Publisher led as Publishers have the direct relationship with the user, and thus the technical and legal basis for processing the user's data.

The browser policies are clear on ability to maintain a cross-domain identity.

- **[Safari](https://webkit.org/tracking-prevention-policy/):** No cross-domain identity (either cookie-based or via finger-printing), first-party identity only, no third-party should be able to create a first-party identity (see limitations placed on client-side storage in ITP 2.1 as "[cross-site trackers have started using first-party sites’ own cookie jars](https://webkit.org/blog/8613/intelligent-tracking-prevention-2-1/)").
- **Firefox:** No cross-domain identity (either cookie-based or via finger-printing), first-party identity only.
- **Chrome:** No cross-domain identity (either cookie-based or via finger-printing), first-party identity only, third-parties can read but not create first-party identity (["Access to the per-first-party identity can be a 3p privilege, not a right"](https://github.com/michaelkleber/privacy-model#third-parties-can-be-allowed-access-to-a-first-party-identity)). Importantly Google, **propose the removal of identity in it's entirety** from ad-tech in both their [FLoCs](https://github.com/jkarlin/floc) and [TURTLEDOVE](https://github.com/michaelkleber/turtledove) proposals.

As a result, in the case of both targeting and frequency capping, under the browsers proposals, these use cases can only be made possible by Publishers. This gives us some broad principles for a privacy-by-design ecosystem:

- Browser policies all indicate that only a Publisher should be capable of creating a first-party identity; third-parties (i.e. the ad-tech ecosystem) should not.
- Browsers are also clear that for a truly privacy safe web, identifiers of any kind should not enter the ad-tech ecosystem (e.g. [FLoCs](https://github.com/jkarlin/floc), [TURTLEDOVE](https://github.com/michaelkleber/turtledove)).
- The GDPR and other regulators are clear that where identity is passed to others, consent must be sought, and a chain is created in the ecosystem.

## An identity based solution

Passing first-party identity into the bid-stream maintains the current status quo within ad-tech, but limits frequency capping to per-domain.

Although the proposal would be the quickest to integrate with the existing ecosystem, it (1) fails to protect user data from leaking and it (2) fails to protect Publisher data from leaking. Importantly, this runs in direct opposition to all proposals put forward by Google's Privacy Sandbox team to date. Engineers on the Webkit team have also been clear on [their dislike for identity in the bid-stream](https://twitter.com/johnwilander/status/1227689494603632640).

A number of precautions could be taken to help protect against the worst outcomes, however ultimately, the system can still be abused by third-parties. We therefore don't believe first-party IDs are a viable solution for frequency capping, but they're still worth documenting.

Some potential solutions:

- **Pass the Publisher's first-party identity into the bid-stream:** The Publisher's first-party ID can be passed directly into the user ID field in the bid request. This is the worst possible outcome for both user privacy and Publisher data leakage. This would allow anyone watching the bid-stream to pick up the user ID and build a profile of the user based upon other data in the bid-request.
- **Pass an encrypted Publisher first-party identity into the bid-stream, which can only be read by trusted parties:** The user ID in the bid request could be encrypted by a Prebid Server using a combination of the Publisher's first-party ID and the bid request's ID or similar. This would ensure that for anyone watching the bid-stream, the user ID is unique for every bid request. Any non-trusted third-party would see a unique user ID for each bid request, despite the bid request originating from the same user. Trusted parties could then be provided with a method for decrypting the user ID (e.g. a decryption key, a decryption web-service hosted on Prebid), such that they can read the Publisher first-party identity. This method would ensure only trusted parties could pick up the user ID, but would not prevent Publisher data from leaking to those parties.
    - **Decryption key for trusted parties:** Provide each trusted party with their own decryption key. This would require a unique bid-request to be created for each DSP. Decryption keys could be periodically rotated to help protect against non-trusted parties gaining access to a decryption key.
    - **Decryption web-service for trusted parties:** Require trusted parties to make a call to a decryption web service hosted by the Publisher (perhaps within Prebid server). This would give Publishers greater transparency into DSPs usage of their first-party identity, but it would come at the cost of having to run highly-available infrastructure.

## An identity-less solution

Today, DSPs implement the frequency capping infrastructure themselves. Doing so however, requires the continuation of identity and prevents a privacy-by-design ecosystem. Passing a Publisher's first-party identity into the bid-stream stands in direct opposite to the proposals and principles laid out in Google's Privacy Sandbox, and similarly, to the direction of regulation like the GDPR.

As a result, we recommend the ecosystem implements an identity-less solutions for frequency capping.

Unlike a solution which requires Publisher's to share their first-party identity alongside user data, an identity-less solution meet's the browsers definitions for privacy—in particular, it meets Google's definitions of privacy-by-design and could be integrated into Google's Privacy Sandbox.

In order to do this, we propose that the Publisher's ad-server (Prebid or similar) manages frequency capping instead of the DSP. In this world, rather than adding a user's ID to each bid-request—thus leaking data and violating a privacy-by-design approach—the DSP specifies the frequency capping logic they want enforced in their bid-response and the Publisher's ad-server executes this logic on their behalf.

Doing so keeps the storage and processing of user data within the Publisher's, and thus data controller's, control. This (1) works to the "privacy by design" principles laid out by the browsers and (2) ensures cleaner compliance under the GDPR and similar regulation (no need to seek consent for DSPs as data controllers).

![Untitled](https://user-images.githubusercontent.com/5756475/93317009-d2e4f780-f804-11ea-8f71-8193a65146ad.png)

Importantly, ad-servers have handled the responsibility for frequency capping on their own domains since their birth. Extending this to the programmatic ecosystem is a natural next step,

Longer term, we think this general theme—a programmatic ecosystem where a Publisher's ad-server is programmable by DSPs—leads to a privacy-by-design ecosystem, unlike today's where the programmatic ecosystem stores and processes user data on third-party clouds. A lack of identity adheres to browsers' privacy-by-design principles (like those in Google's Privacy Sandbox) and minimises privacy risk in the supply chain (thus more robust and safe under the GDPR and similar).

## User Agent Support

We believe that ultimately Frequency Capping should be handled by the browser. Google's Turtledove proposal already describes a solution for the interest group request, which allows the DSP to tell the browser its Frequency Capping rules:

![Untitled 1](https://user-images.githubusercontent.com/5756475/93316979-c791cc00-f804-11ea-9b52-9b244e72a711.png)

Extending this to the "*contextual ad request*" would allow on-device frequency capping as described above, but cross-domain. While implementing the logic on a solution level (header bidding wrapper, SSP, ad server) will be limited to same-domain Frequency Capping, it allows ad-tech to
1. move faster and make a case for browser to support this functionality natively
2. implement a fall-back for browsers not supporting Frequency capping natively
3. offer a solution for cookie-less environments today (Safari, Firefox, Brave)