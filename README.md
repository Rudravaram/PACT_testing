# PACT_testing

PACT Testing, or Consumer-Driven Contract Testing, is a technique used to ensure that services (such as microservices) interact with each other correctly. It focuses on testing the interactions between services by defining and verifying contracts between them.

<img src="https://github.com/user-attachments/assets/47eb9ab1-eb3f-4279-9bb0-58cf4e6838bc" alt="Landscape Image 2">

### Key Concepts of PACT Testing

1. **Consumer and Provider**:
   - **Consumer**: The service or client that makes requests to another service.
   - **Provider**: The service that responds to the consumer's requests.

2. **Contract**:
   - A contract is an agreement between the consumer and provider that outlines the expected interactions. It includes details like request and response formats.

3. **Pacts**:
   - Pacts are the specific agreements (contracts) created by the consumer tests and verified by the provider.

### Workflow of PACT Testing

1. **Define Contracts**:
   - The consumer creates a set of expectations (pacts) for the interactions it has with the provider.

2. **Verify Contracts**:
   - The provider tests itself against the pacts defined by the consumer to ensure it meets the consumer's expectations.

3. **Publish Pacts**:
   - Pacts are published to a pact broker, which serves as a repository for pacts and can facilitate collaboration between teams.

4. **Continuous Integration**:
   - Pacts are verified as part of the CI/CD pipeline to ensure that changes do not break existing contracts.

### Benefits of PACT Testing

- **Early Detection of Issues**: Problems in the interaction between services are caught early in the development process.
- **Decoupling**: Services can be developed and tested independently as long as they adhere to the contract.
- **Clear Documentation**: Contracts serve as clear documentation of service interactions.

### Implementing PACT Testing

To implement PACT testing, you can follow these steps:

1. **Set Up Your Project**:
   - Choose a PACT library for your language (e.g., Pact JVM for Java, Pact JS for JavaScript, etc.).

2. **Consumer-Side Testing**:
   - Write tests to define the expected interactions with the provider.
   - These tests generate pacts, which are then stored (usually in JSON format).

3. **Provider-Side Testing**:
   - Write tests to verify that the provider adheres to the pacts.
   - These tests ensure the provider meets the consumer's expectations.

4. **Publishing and Verification**:
   - Use a pact broker to publish and share pacts.
   - Integrate pact verification into your CI/CD pipeline to automate the testing process.

### Example in JavaScript

Here’s a brief example of implementing PACT testing in JavaScript using `pact-js`:

**Step 1: Install Dependencies**

```bash
npm install @pact-foundation/pact
```

**Step 2: Create Consumer Test**

```javascript
const { Pact } = require('@pact-foundation/pact');
const path = require('path');
const axios = require('axios');

// Define the consumer test
describe('Pact with Provider', () => {
  const provider = new Pact({
    consumer: 'ConsumerService',
    provider: 'ProviderService',
    port: 1234,
    log: path.resolve(process.cwd(), 'logs', 'pact.log'),
    dir: path.resolve(process.cwd(), 'pacts'),
    logLevel: 'INFO'
  });

  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());

  it('should receive the expected response from the provider', async () => {
    // Define the interaction
    await provider.addInteraction({
      uponReceiving: 'a request for data',
      withRequest: {
        method: 'GET',
        path: '/data',
        headers: { 'Accept': 'application/json' }
      },
      willRespondWith: {
        status: 200,
        headers: { 'Content-Type': 'application/json' },
        body: { key: 'value' }
      }
    });

    // Make the request and verify the response
    const response = await axios.get('http://localhost:1234/data');
    expect(response.data).toEqual({ key: 'value' });

    // Verify the interactions
    await provider.verify();
  });
});
```

**Step 3: Provider Verification**

```javascript
const { Verifier } = require('@pact-foundation/pact');
const path = require('path');

describe('Pact Verification', () => {
  it('should validate the expectations of the consumer', async () => {
    const opts = {
      providerBaseUrl: 'http://localhost:8080',
      pactUrls: [path.resolve(process.cwd(), 'pacts', 'consumer-service-provider-service.json')]
    };

    await new Verifier().verifyProvider(opts);
  });
});
```

### Conclusion

PACT testing is an effective way to ensure that services interact correctly by defining and verifying contracts between them. Implementing PACT testing involves setting up consumer and provider tests, using a pact broker to share pacts, and integrating these tests into your CI/CD pipeline. This approach helps in detecting issues early, decoupling services, and providing clear documentation of interactions.
