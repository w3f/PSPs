# Polkadot Contracts Proposals (PCPs)


A Polkadot Contracts Proposals (PCPs) describes standards for the Polkadot ecosystem. The Polkadot Standards Proposal GitHub is a community-based initiative.  
PCPs process is not supposed to be a substitute for the Polkadot Governance process and is meant to focus only on commonly agreed usage patterns rather than protocol adjustments.  

> __Disclaimer__: The Polkadot Standards Proposals (PSPs) have been renamed to Polkadot Contracts Proposals (PCPs). Everything else should be discussed as part of the [RFCs process](https://github.com/polkadot-fellows/RFCs).  

---

- [Polkadot Contracts Proposals (PCPs)](#polkadot-standards-proposals-pcps)
  - [:clipboard: Process](#clipboard-process)
  - [:pencil: Contributing](#pencil-contributing)
  - [:bulb: Help](#bulb-help)

## :clipboard: Process  

Below is the workflow of a successful PCPs:
```
1. Draft -> 2. Call for Feedback -> 3. Published -> 4. Integrated
```
1. **Draft:** A valid draft, which is merged into the [draft
   subfolder](./PCPs/drafts) and actively improved together with the community.
2. **Call for Feedback:** The PCP will be shared on different public channels for
   additional feedback for at least two weeks. The result of this step is either
   an acceptance of the standard (->Published) or a rejection (->Draft).
3. **Published:** Any further changes are unlikely, and developers can start
   integrating the PCP.
4. **Integrated:** The PCP is actively used, and a reference implementation
   exists.

In order to be **merged or accepted** for the different stages, reviewers need to approve a PR. Reviewers should be known experts in the topic covered by the PCP. 

## :pencil: Contributing

Before you start writing a formal PCP, you should discuss an idea in the various community public channels (see the [Polkadot community website](https://polkadot.network/community/)). A PCP should provide the motivation as well as a technical specification for the feature. 

1. Fork this repository.
2. In the newly created fork, create a copy of the template.
3. Fill out the [template](./PCPs/pcp-template.md) with the details of your PCP. If your PCP requires images, the images should be integrated into a subdirectory of the src folder, which has your PCP number as the name.
4. Once you have completed the application, click on "create new pull request".
5. Rename the file with "pcp-number_of_your_pr.md".
6. Update the pull request. 

## :bulb: Help

* [GitHub Discussions](https://github.com/w3f/PCPs/discussions)
* [PCP Channel on Element](https://app.element.io/#/room/#psp:web3.foundation)
