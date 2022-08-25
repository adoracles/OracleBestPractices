# Best Practices for Oracle Users
 
Oracles provide data on-chain. Different oracles accomplish this in different ways and your oracle design should be based on the needs of your application or protocol. However, the data definition, sources, aggreation, and how the oracle is implemented in your smart contracts can follow some best practices to ensure a secure oracle implementation. 

## Defining your data
No matter which oracle you decide is best for your use case, you will have to provide a basic data specification so that the oracle provides the data your protocol requires. The main questions you have to answer are:  what do you need? from where? should it be aggregated?

 
### *What data/information do you need*
The first step on obtaining a secure data feed is to define the data you need. You can be as specific or vague as your protocol needs. For example, you can just ask for the volume weighted average price of ETH on the top 10 exchanges (or who won the superbowl in 2022). You can also be more specific by asking for the 24H volume weighted average price of ETH at 10AM for your specified 10 exchanges and provide the specific APIs that should be used. The advantage of being specific is that the difference between any reporter providing the data will be decreased. On the other hand, a disadvantage of being so specific is that you have to plan for data unavailability (what happens if one of the API is down? or one of the exchanges loses liquidity?). When the definition is vague it can be more dynamic and more censorship resistant but it comes with a higher degree of volatility because the data will depend on the data reporter's choice of data sources.  These are some trade-offs you need to keep in mind as you provide a data definition to the oracle.

 
### *Choosing data sources*
As a general rule, if you are specifying the data sources, provide 3 or more data sources. Denial of service (DDOS) attacks are probably cheaper than breaking an oracle protocol so having multiple sources and fallbacks for these helps lower the risk of this kind of attack. Also, if you choose a set of paid API’s keep in mind that the cost to incentivize reporters to pay for the API and gas fees can be passed down to the user. Paid APIs are also, generally, centralized, so a fallback should be included in the event of censorship or data unavailability. If you are using APIs from several exchanges that is great but what happens if the exchange goes down? Or the liquidity for the asset becomes so low it becomes very cheap to manipulate the price? Choose your data sources wisely and have a contingency plan built into your protocol design.

 
### *Choosing aggregation*
Determine what data aggregations you need (median, mean, 24h volume weighted averages, etc…). Not all aggregations are created equal. The mean of 2 spot prices can be easier to manipulate than a 24h volume weighted average. If you are requesting price data, is the asset liquid enough or can the price be easily manipulated?  For non-numeric data, you may have to get creative with the way the data is requested or answered or just allow enough time before executing based on it. Ultimately, you have to determine what your protocol needs.


 
## Choosing an appropriate oracle(s)
Your protocol may be an idea right now but projects have the potential to grow exponentially in crypto so when determining which oracle or group of oracles to use, keep security in mind. If you have a goal in mind for funds secured by your protocol, you can compare that with the cost to break or grief different oracles. That calculation can help you determine if you need to use more than one oracle or if your system should use a primary oracle and a fallback oracle. The goal is to maximize the cost to break your data feed/oracle implementation. Different oracles have different attack vectors, and this alone can make it very difficult and expensive to break your data feed. The incentive to break will always be there when potential gains or losses are larger than the cost to break the oracle but with proper implementation and design, you can ensure that threshold is as high as possible. Unfortutately, there is no one oracle design and easy way to calculate the cost to break for all of them since they work differently. 

DRAFT TODO: Can each oracle provide a formula to calculate the cost to grief their protocol for 1 hour?
 
Here is a YouTube series where the founders or people very familiar with these oracle protocols explain how they work: [Oracle deep dive](https://www.youtube.com/channel/UCtFzhqGOVXyi91gaiIBEkNw). However, the best way to learn about a protocol is to dive into their documentation, their github, and ask questions on their discord or telegram channels.

 
## Smart architecture for your oracle
Once you determine what oracle or group of oracles to use, you can design the best architecture for the implementation and plan for the worst case scenario. Here are some examples of the architecture considerations when designing your oracle implementation: 
 
- How will the oracle be implemented in your protocol? Meaning will you use a median of at least three oracles? or are there primary, secondary, and fallback oracles in your system? and how are the fallbacks triggered?
- What happens if no data is received?
- What happens if bad data gets through (the oracle or data source is compromised)? How would your protocol or community handle that?
- How would your protocol or users incentivize the oracle reporters to provide data on-chain? Do you build in a fee or subsidize the cost?
- Have enough confirmations been received before the data is used to execute? 

Your oracle cannot be faster than the chain and it cannot reach finality before the chain reaches finality. It is best practice to allow for several chain confirmations (similar to how big exchanges wait 7-10 confirmations before updating your balance) before using data to execute on your smart contract.
 

## Simple code checks before executing
If you have kept these best practices in mind through your design then the code implementation should be fairly simple. But here are a few more best practice checks we recommend you implement in your code such as checking if the data exists(was provided by your oracle), if the oracle’s latest value is current and that it is reasonable enough. This can be easily accomplished with a few require statements in solidity or similar in other languages.
 
- Check that a value was provided by the oracle.

```javascript
    (bool ifRetrieve, bytes memory _value, ) =  getCurrentValue(_queryId);
    	if (!ifRetrieve) return "0x";
    	return _value;
```

- Check that the value is current enough but that it allowed enough time to be safe to use. You will have to determine what is “current” for your protocol. The example below checks that the data is at least one hour old to allow for several confirmations but not older than a day.

```javascript
    require (now – timestamp < 1 day  && now-timestampt > 1 hour)
```

- Check that the the data reasonable. This example checks that the current value is not more than 40 percent of the previous value.

DRAFT TODO: from decimal to integer since there are no decimals in solidity.

```javascript
    require abs(int256 current val – prev val ) < prev val * .40
 ```

<br>

## Conclusion
Although this guide may seem simple, following these guidelines will help you design a more secure oracle implementation. Your protocol may be flawless and the oracle may have been battle tested but the security of a protocol that depends on oracle data to execute lies on a secure and smart implementation of the oracle. Happy building!
 

 <br>
 <br>
 
DRAFT TODO: This could be a checklist

#### Best practices TLDR

- Define your data
    * Determine what data you need
    * Find three or more data sources for your data
    * Choose your aggregation
- Choose your oracle(s)
    * The cost to break/grief your oracle implementation should be greater than the value secured
- Smart architecture for your oracle
    * Choose more than one oracle or use primary, secondary, etc... fallback methods and specify triggers
    * Plan for the worse (what happens if there is no data? Bad data? or reporters are not incentivized to provide the data?)
    * Allow a wait period between when the oracle data is received and used (wait for several confirmations before executing and allow time for data validity disputes)
- Simple code checks before executing
    * Check that a value was provided by the oracle
    * Check that the value is current enough
    * If needed, check that the value is within a range of the previous value (this depends on
the asset)
