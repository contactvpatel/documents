### Developer Guideline for Using Specific Environments  

This guideline outlines the best practices for utilizing environments while developing and testing **Core Applications** and **Business Applications**. Following these practices ensures stability, alignment with production features, and minimizes disruption during the development lifecycle.  

#### **Core Applications**
Core Applications include **SSO**, **MIS**, **MDS**, **ASM**, and others.  
- **Environment Details**:
  - **Dev Environment**: Actively used for feature development. This environment is **unstable** and may contain incomplete or experimental changes.
  - **QA Environment**: Used for testing the newly developed features. This environment is also **unstable** as it contains features not yet deployed to Production.  
  - **UAT Environment**: Simulates the Production environment. Features in this environment are thoroughly tested and are closer to being deployed to Production.  

- **Development Lifecycle**:  
  Core Applications have their own **independent development and release cycle**. As a result, their Dev and QA environments frequently undergo changes that may not be compatible with Business Applications.  

#### **Business Applications**
Business Applications include **EMS**, **ERS**, **ALM**, and others.  

- **Environment Usage**:
  - Avoid using **Core Application Dev or QA environments** for Business Application development or testing. These environments are **unstable** and may contain changes not aligned with the production environment.  
  - Always use **Core Application’s UAT environment** for Business Application development and testing. The UAT environment is **stable** and reflects the Production state of Core Applications, ensuring compatibility and reliability.  

- **Reasoning**:  
  - Business Applications often depend on stable features of Core Applications. Using unstable environments like Dev or QA can lead to mismatched functionality and wasted debugging efforts.  
  - Core Application UAT environment provides a reliable baseline that closely mirrors Production.  

#### **Guidelines for Development and Testing**  

1. **Core Application Development**:  
   - Use **Core Application Dev** for feature development and dev testing.  
   - Conduct feature and QA testing in the **Core Application QA** environment.  
   - Deploy thoroughly QA tested features to the **Core Application UAT** environment for final UAT testing.  

2. **Business Application Development**:  
   - When integrating with Core Applications, always point to the **Core Application UAT environment**.  
   - Avoid using Core Application Dev or QA environments for development or testing.  
   - Conduct your integration and end-to-end testing in your application’s designated UAT environment, pointing to Core Application UAT.  

3. **Pre-Production Testing**:  
   - For critical validation before release, ensure both Core and Business Applications point to their respective **UAT environments**.  
   - Perform smoke tests to confirm that the functionality aligns with the expected Production behavior.  

#### **Communication and Coordination**  
- Notify the Core Application team of any critical dependencies or features required in the UAT environment.  
- Maintain proper documentation of version compatibility between Core and Business Applications.  
- Schedule regular sync-ups to address any dependency updates or changes.

Here’s a table outline listing the UAT environment details for **SSO**, **MIS**, and **MDS**, including their UI and API links:

| **Application** | **Environment** | **UI Link**                | **API Link**                  |
|------------------|-----------------|----------------------------|-------------------------------|
| **SSO**         | UAT             | https://sso.uat.na.bapsapps.org/  | https://api.uat.bapsapps.org/sso/api/          |
| **MIS**         | UAT             | https://uat.baps.dev/mis          | https://api.uat.bapsapps.org/mis/api/          |
| **MDS**         | UAT             | https://uat.baps.dev/mds          | https://api.uat.bapsapps.org/mds/api/          |
| **ASM**         | UAT             | https://uat.baps.dev/asm          | https://api.uat.bapsapps.org/asm/api/          |

#### **Conclusion**  
By adhering to this guideline, developers can ensure smooth integration between Core and Business Applications, reduce risks from unstable environments, and maintain alignment with production-ready features. Always prioritize stability and compatibility in shared environments.
