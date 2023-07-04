Different DID methods exhibit distinct characteristics and involve inherent design trade-offs. For example, some methods prioritize instant DID creation but lack mechanisms for subsequent updates, while others prioritize high throughput of DID events but sacrifice features like DID exchange or historical auditability.

When choosing a DID method that aligns with specific use cases, it is crucial to thoroughly evaluate the associated trade-offs. It is hard to expect any single DID method to cater to all possible scenarios, and the PRISM DID method is no exception. Therefore, we list the principles and trade-offs of our DID method in the next section.

## Trade-offs for PRISM DID method

The PRISM DID method follows specific principles and trade-offs, which are important to consider. These principles include:
1. Strong Availability: All update and deactivation events in the PRISM DID method are posted on-chain, ensuring public accessibility. Once published, these events cannot be deleted from the blockchain, enabling historical auditability.
1. Decentralization: PRISM allows anyone to create, resolve, and manage their DIDs without requiring authorization from external parties. This ensures censorship-resistant actions and promotes decentralization.
1. Instant Creation: PRISM enables instant DID creation without requiring interaction with the blockchain, ensuring a seamless user experience.
1. DID Transferability: Due to the visibility and immutability of updates, PRISM DIDs support the exchange of ownership.

### Current Limitations

The PRISM DID method has certain limitations that should be considered, including:
- Latency: Performing DID update or deactivation events requires submitting a blockchain transaction, which introduces a delay between submission and resolution.
- Throughput Limit: Posting events on-chain is subject to the space limits imposed by transaction and block sizes, potentially affecting the method's throughput.

## Steps to request a new feature

When requesting new features for the PRISM DID method, it is important to ensure alignment with the method's design principles. Consider the following steps:
1. Describe the problem you aim to solve, focusing on the underlying use case rather than proposing the feature itself.
1. Clearly outline the intended new feature and its functionality.
1. Explain why your use cases cannot be accomplished without the proposed feature or, if possible, describe the trade-offs between the new feature and the current approach.
1. Suggest potential implementation paths for the new feature.
1. Justify how the new feature aligns with the principles of the PRISM DID method and demonstrate that it maintains the method's key properties without breaking them.

By following these steps, you can effectively initiate a conversation around requesting new features for the PRISM DID method, ensuring that they enhance functionality and mitigate existing trade-offs while adhering to the method's core principles.
