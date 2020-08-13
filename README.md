# Solidity

#				                    ********** HomeWork 20 **********

### The Profit Splitter contract is on the JavaScript VM environment using three accounts to send wei back and forth. 
### It enables you to send money to employees  without worrying about sending more money than you have to send.
### You do this by using the deposit function and inputting how much you want to deposit,
### and then using the getbalance function to make sure the deposit was valid.



### You can also use the tierred ProfitSplitter contract to send send money to employees adding a distribution percentage,
### this helps when multiple people own part of the company. This will make sure no funds are unaccounted for,
### and all the money is distributed correctly.


### The DeferredEquitPlan distributes shares of a company.
### It will check to make sure the account is active before sending the money to ensure no funds are lost in the transaction.
### No contract will accept ether and can fast forward to see eventual account balances.
### By deploying the contract and using the fastforward function to simulate time and distribute shares, 
### you can then use the distributed_shares tab to see how many shares were distributed to the account.





##							********** AssociateProfitSplitter **********


pragma solidity ^0.5.0;

// lvl 1: equal split
contract AssociateProfitSplitter {
    // @TODO: Create three payable addresses representing `employee_one`, `employee_two` and `employee_three`.
    address payable employee_one;
    address payable employee_two;
    address payable employee_three;
    
    address public last_to_withdraw;
    uint public last_withdraw_block;
    uint public last_withdraw_amount;

    address public last_to_deposit;
    uint public last_deposit_block;
    uint public last_deposit_amount;

    uint unlock_time;

    uint fakenow = now;
    
      // Constructor for setting the owner of the contract as the one who deploys the contract
    constructor(address payable _one, address payable _two, address payable _three) public {
        employee_one = _one;
        employee_two = _two;
        employee_three = _three;
    }


  // Obtains the balance residing within the contract
    function getBalance() public view returns(uint) {
      return address(this).balance;
    }
  
  
    function deposit() public payable {
        // @TODO: Split `msg.value` into three
        // Your code here!
        uint amount = msg.value / 3;


        // @TODO: Transfer the amount to each employee
        // Your code here!
        msg.sender.transfer(amount);


        // @TODO: take care of a potential remainder by sending back to HR (`msg.sender`)
        // Your code here!
        if (last_to_deposit != msg.sender) {
      last_to_deposit = msg.sender;
    }

    last_deposit_block = block.number;
    last_deposit_amount = (msg.value - amount * 3);
    }    
    

    function() external payable {deposit();}
        // @TODO: Enforce that the `deposit` function is called in the fallback function!
        // Your code here!
}






##						********** Tiered Profit Splitter **********




pragma solidity ^0.5.0;

// lvl 2: tiered split
contract TieredProfitSplitter {
    address payable employee_one; // ceo
    address payable employee_two; // cto
    address payable employee_three; // bob

    constructor(address payable _one, address payable _two, address payable _three) public {
        employee_one = _one;
        employee_two = _two;
        employee_three = _three;
    }

    // Should always return 0! Use this to test your `deposit` function's logic
        function balance() public view returns(uint) {
            return address(this).balance;
    }

        function deposit() public payable {
        uint points = msg.value / 100; // Calculates rudimentary percentage by dividing msg.value into 100 units
        uint total;
        uint amount;

        // @TODO: Calculate and transfer the distribution percentage
        
        // Step 1: Set amount to equal `points` * the number of percentage points for this employee
        amount = (points * 60);
        employee_one.transfer(amount);
        
        // Step 2: Add the `amount` to `total` to keep a running total
        total += amount;
        
        // Step 3: Transfer the `amount` to the employee
        

        // @TODO: Repeat the previous steps for `employee_two` and `employee_three`
        // Your code here!
        amount = (points * 25);
        employee_two.transfer(amount);
        
        amount = (points * 15);
        employee_three.transfer(amount);
        

        employee_one.transfer(msg.value - total); // ceo gets the remaining wei
    }

        function() external payable {
        deposit();
        }
}




##						********** Deferred Equity Plan **********


pragma solidity ^0.5.0;
// lvl 3: equity plan
contract DeferredEquityPlan {
    uint fakenow = now;
    address human_resources;

    address payable employee; // bob
    bool active = true; // this employee is active at the start of the contract

    // @TODO: Set the total shares and annual distribution
    // Your code here!
    uint total_shares = 1000;
    uint annual_distribution = 250;

    uint start_time = fakenow; // permanently store the time this contract was initialized

    // @TODO: Set the `unlock_time` to be 365 days from now
    // Your code here!
    uint unlock_time = (fakenow + 365 days);

    uint public distributed_shares; // starts at 0

    constructor(address payable _employee) public {
        human_resources = msg.sender;
        employee = _employee;
    }

    function distribute() public {
        require(msg.sender == human_resources || msg.sender == employee, "You are not authorized to execute this contract.");
        require(active == true, "Contract not active.");
        
        // @TODO: Add "require" statements to enforce that:
        // 1: `unlock_time` is less than or equal to `now`
        require(unlock_time <= fakenow, "Invalid Unlock Time");
        
        // 2: `distributed_shares` is less than the `total_shares`
        require(distributed_shares < total_shares, "Not Enough Shares");
        
        // Your code here!

        // @TODO: Add 365 days to the `unlock_time`
        // Your code here!
        unlock_time = fakenow + 365 days; 

        // @TODO: Calculate the shares distributed by using the function (now - start_time) / 365 days * the annual distribution
        // Make sure to include the parenthesis around (now - start_time) to get accurate results!
        // Your code here!
        distributed_shares = (fakenow - start_time) / 365 days * annual_distribution;

        // double check in case the employee does not cash out until after 5+ years
        if (distributed_shares > 1000) {
            distributed_shares = 1000;
        }
    }

    // human_resources and the employee can deactivate this contract at-will
    function deactivate() public {
        require(msg.sender == human_resources || msg.sender == employee, "You are not authorized to deactivate this contract.");
        active = false;
    }

    // Since we do not need to handle Ether in this contract, revert any Ether sent to the contract directly
    function() external payable {
        revert("Do not send Ether to this contract!");
    }
    function fastforward() public {
    fakenow += 100 days;
}

}