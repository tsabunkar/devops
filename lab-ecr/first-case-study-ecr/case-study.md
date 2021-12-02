Case Study – Business Requirement

Microservices – Business Requirement

Contoso Industrial Development Bank (CIDB) is a fictitious banking corporation, which was started in the year 1996, which has large base customers in the country, and has numerous branches across the country and has been processing all types of regular banking transactional activities, loan requests and credit card transactions to their customers. As per the current requirement, they are required to now modernize the application and its architecture for credit card enrolment of their new and existing customers.

Whenever a customer wants to ask for a new Credit Card, they will enter a bunch of data through some Forms. This data will get to a Card Service that will create the data needed to create Digital Credit Card on the fly (in the meantime the actual physical card gets processed and sent). For this purpose, it will first check the provided customer’s data is accurate and can be validated through the Identity Service (maybe this service will use a third-party service like CIBIL to check the customer’s identity). Then, the Calculation Service will determine the credit limit for the new credit card based on internal calculations and other factors. Once everything is successfully completed the Card Service can finally return all the data for the new Digital Credit Card to the customer that was computed in the beginning.

Use Case

The best way to describe this is to show the diagrams of the sequences of transactions happening just after the customer has entered all the required data in the form.

The input data will get to the Card Service that will create a new record in the Card DB. Note we have a status value, which will play a key role here. For now, it is set with VERIFYING value.

Once the record is successfully persisted, a new event will be streamed by Card Service and consumed by Identity Service.

Identity service will consume the event, then it will execute its internal logic and persist a new record in the Identity DB. The is Verified attribute will be a Boolean stating if the identity was indeed verified or not.

Once the record is successfully persisted, a new event will be streamed and will be consumed by Card Service and Calculation Service.

The event consumed by Card Service will trigger an update of the record persisted in the first step. The status value will be updated to IDENTITY_VERIFIED. We are notifying the main Service that the identity was verified.

The event will also be consumed by Calculation Service. This one will execute some internal logic so that the goal is to calculate the amount Limit for the potential new Credit Card and persist this new data to a new record in the Amount DB.

Once the record is successfully persisted a new event with the amount limit data will be streamed to be consumed by the Card Service.

Finally, once the event is consumed by the Card Service, we will again update the initial record with its final state. The amount Limit value will be updated, and the status will change to VERIFIED. This status value means the new Credit Card Data is ready to be returned to the customer so they can start using their new Digital Credit Card.

![case-study-ecr](./first-case-study-ecr.png 'case-study-ecr')
