# Akbank-Practicum-Hw3
Build and Deploy a Crowdfund Application

```Solidity
pragma solidity ^0.8.7;
//SPDX-License-Identifier: MIT

//This IERC20 Import is out of date
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
contract CrowFund {
    event Launch (
        uint id,
        address indexed creator,
        uint goal,
        uint32 startAt,
        uint32 endtAt
    );
    event Cancel(
        uint _id
    );
    event Pledge (
        uint indexed id,
        address indexed caller,
        uint amount
    );
    event UnPledge (
        uint indexed id,
        address indexed caller,
        uint amount
    );
    event Claim(uint _id);
    event Refund(uint indexed id, address indexed caller, uint amount);
    

    struct Campaign {
        address creator;
        uint goal;
        uint pledged;
        uint32 startAt;
        uint32 endtAt;
        bool claimed;
    }

    IERC20 public immutable token;
    uint public count;
    mapping(uint => Campaign) public campaigns;
    mapping(uint => mapping(address => uint)) public pledgedAmount;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function launch(
        uint _goal,
        uint32 _startAt,
        uint32 _endAt
    ) external {
        require(_startAt >= block.timestamp, "start at < now");
        require(_endAt >= _startAt, "end at < start at");
        require(_endAt <= block.timestamp + 90 days, "end at > max duration");

        count += 1;
        campaigns[count] = Campaign({
            creator: msg.sender,
            goal: _goal,
            pledged: 0,
            startAt: _startAt,
            endtAt: _endAt,
            claimed: false

        });

        emit Launch(count, msg.sender, _goal, _startAt, _endAt);
    }

    function cancel(uint _id) external {
        Campaign memory campaign = campaigns[_id];
        require(msg.sender == campaign.creator, "not creator");
        require(block.timestamp < campaign.startAt, "started");
        delete campaigns[_id];
        emit Cancel(_id);
    }

    function pledge(uint _id, uint _amount) external {
        Campaign storage campaign = campaigns[_id];
        require(block.timestamp >= campaign.startAt,"not started");
        require(block.timestamp <= campaign.endtAt,"ended");

        campaign.pledged += _amount;
        pledgedAmount[_id][msg.sender] += _amount;
        token.transferFrom(msg.sender, address(this), _amount);

        emit Pledge(_id, msg.sender, _amount);
    }

     function unpledge(uint _id, uint _amount) external {
        Campaign storage campaign = campaigns[_id];
        require(block.timestamp <= campaign.endtAt,"ended");

        campaign.pledged -= _amount;
        pledgedAmount[_id][msg.sender] -= _amount;
        token.transfer(msg.sender, _amount);

        emit UnPledge(_id, msg.sender, _amount);
    }

    function claim(uint _id) external {
        Campaign storage campaign = campaigns[_id];
        require(msg.sender == campaign.creator, "not creator");
        require(block.timestamp > campaign.endtAt, "not ended");
        require(campaign.pledged >= campaign.goal, "pledged < goal");
        require(!campaign.claimed, "claimed");
        
        campaign.claimed = true;
        token.transfer(msg.sender, campaign.pledged);

        emit Claim(_id);
    }
    function refund(uint _id) external {
    Campaign storage campaign = campaigns[_id];
    require(block.timestamp > campaign.endtAt,"not ended");
    require(campaign.pledged < campaign.goal,"pledged < goal");

    uint bal = pledgedAmount[_id][msg.sender];
    pledgedAmount[_id][msg.sender] = 0;
    token.transfer(msg.sender, bal);
    

    emit Refund(_id, msg.sender,bal);
}

}
```
