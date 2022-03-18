# Polkadot Standards Proposals (PSPs)


A Polkadot Standards Proposal (PSP) describes standards for the Polkadot ecosystem. The Polkadot Standards Proposal GitHub is a community-based initiative.  
PSP process is not supposed to be a substitute for Polkadot Governance process and is meant to focus only on commonly agreed usage patterns rather than protocol adjustments.  

> __Disclaimer__: The Polkadot network is relatively young and many ecosystem
projects are just getting started. While the interoperability aspect of
Polkadot, parachains and community tools are still being actively developed,
participants of the ecosystem can create proposals for certain mechanisms for
the wider community to use. As of now, we loosely accept proposals that we do
not actively endorse or expect to be implemented. We think many standards will
be adopted organically (with some coordination) and change with time. We expect
that a more firm process for standardization will evolve as adoption takes
place. Certain proposals might be adjusted, replaced or deprecated.

---

- [Polkadot Standards Proposals (PSPs)](#polkadot-standards-proposals-psps)
  - [:clipboard: Process](#clipboard-process)
  - [:pencil: Contributing](#pencil-contributing)
  - [:bulb: Help](#bulb-help)

## :clipboard: Process  

Below is the workflow of a successful PSP:
```
1. Draft -> 2. Call for Feedback -> 3. Published -> 4. Integrated
```
1. **Draft:** A valid draft, which is merged into into the [draft
   subfolder](./PSPs/drafts) and actively improved together with the community.
2. **Call for Feedback:** The PSP will be shared on different channels for
   additional feedback for at least 2 weeks. The result of this step is either
   an acceptance of the standard (->Published) or the rejection (->Draft).
3. **Published:** Any further changes are unlikely, and developers can start
   integrating the PSP.
4. **Integrated:** The PSP is actively used and a reference implementation
   exists.

In order to be **merged or accepted** for the different stages, reviewers need to approve a PR. Reviewers should be known experts in the topic covered by the PSP. 

## :pencil: Contributing

Before you start writing a formal PSP, you should discuss an idea in the various community channels (see the [Polkadot community website](https://polkadot.network/community/)). A PSP should provide the motivation as well as a technical specification for the feature. 

1. Fork this repository.
2. In the newly created fork, create a copy of the template.
3. Fill out the [template](./PSPs/psp-template.md) with the details of your PSP. If your PSP requires images, the images should be integrated in a subdirectory of the src folder, which has your PSP number as the name.
4. Once you have completed the application, click on "create new pull request".
5. Rename the file with "psp-number_of_your_pr.md".
6. Update the pull request. 

## :bulb: Help

* [GitHub Discussions](https://github.com/w3f/PSPs/discussions)
* [PSP Channel on Element](https://app.element.io/#/room/#psp:web3.foundation)
