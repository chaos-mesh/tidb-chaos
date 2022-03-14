# Format of records

The record for each test scenario should contain the content below:

1. Description

   A brief description of the test scenario. For example:

   ```text
   Test the availability of PD in network partition scenarios.
   ```

2. Hypothesis

   Describes the expected behavior of the TiDB cluster under failure. For example:

   ```text
   When a few PD nodes suffered network partitions, the TiDB cluster can still provide services normally.
   ```

3. Process

   Describe the specific process of the experiment, including how to configure faults, how to inject faults, and how to verify the results. For example:

   ```text
   1. Chaos Mesh fault YAML configuration

   ......

   2. Using Kubectl to create the experiment

   ......

   3. Verifying TiDB's status

   ......

   ```

4. Result

   Judge whether the hypothesis is correct or not based on the results of the test process. For example:

   ```text
   After injecting network partition into PD, TiDB can still write data normally. So the hypothesis is correct.
   ```
