# Emergency-Withdrawal-Pausable-Fund
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract EmergencyFund {
    address public owner;
    bool public paused;
    uint256 public emergencyDelay;
    uint256 public emergencyRequestTime;

    error NotOwner();
    error ContractPaused();
    error EmergencyNotRequested();
    error TooEarly();

    event EmergencyRequested();
    event EmergencyWithdrawn(uint256 amount);

    constructor(uint256 _delay) {
        owner = msg.sender;
        emergencyDelay = _delay;
    }

    modifier onlyOwner() {
        if (msg.sender != owner) revert NotOwner();
        _;
    }

    modifier whenNotPaused() {
        if (paused) revert ContractPaused();
        _;
    }

    function requestEmergencyWithdraw() public onlyOwner {
        emergencyRequestTime = block.timestamp;
        emit EmergencyRequested();
    }

    function emergencyWithdraw() public onlyOwner {
        if (emergencyRequestTime == 0) revert EmergencyNotRequested();
        if (block.timestamp < emergencyRequestTime + emergencyDelay) revert TooEarly();

        uint256 amount = address(this).balance;
        paused = true;
        payable(owner).transfer(amount);
        emit EmergencyWithdrawn(amount);
    }

    receive() external payable {}
}
