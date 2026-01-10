graph TD
    subgraph HR_System ["HRIS (Workday/Oracle)"]
        A[HR Event: New Hire/Transfer/Termination]
    end

    subgraph IGA_Engine ["Identity Governance (SailPoint)"]
        B{Event Type?}
        C[Joiner: Birthright Provisioning]
        D[Mover: Delta Access Check]
        E[Leaver: Immediate De-provisioning]
    end

    subgraph Targets ["Downstream Systems"]
        F[Active Directory / Azure AD]
        G[SaaS Apps: Slack, Salesforce]
        H[CyberArk: Privileged Accounts]
    end

    A --> B
    B -- New Hire --> C
    B -- Role Change --> D
    B -- Exit --> E

    C --> F & G
    D --> |Add/Remove| F & G
    E --> |Disable| F & G & H

    D --> |High Risk Role?| H
