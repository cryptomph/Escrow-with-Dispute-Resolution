# Escrow-with-Dispute-Resolution
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/Ownable.sol";

contract SecureEscrow is Ownable {
    enum Status { Pending, Funded, Released, Disputed, Refunded }

    struct Deal {
        address buyer;
        address seller;
        uint256 amount;
        Status status;
    }

    mapping(uint256 => Deal) public deals;
    uint256 public dealCounter;

    event DealCreated(uint256 dealId);
    event FundsReleased(uint256 dealId);
    event DisputeRaised(uint256 dealId);

    function createDeal(address seller) external payable {
        require(msg.value > 0, "Send funds");
        dealCounter++;
        deals[dealCounter] = Deal(msg.sender, seller, msg.value, Status.Funded);
        emit DealCreated(dealCounter);
    }

    function releaseFunds(uint256 dealId) external {
        Deal storage deal = deals[dealId];
        require(msg.sender == deal.buyer || msg.sender == owner(), "Not authorized");
        require(deal.status == Status.Funded, "Cannot release");
        deal.status = Status.Released;
        payable(deal.seller).transfer(deal.amount);
        emit FundsReleased(dealId);
    }

    function raiseDispute(uint256 dealId) external {
        Deal storage deal = deals[dealId];
        require(msg.sender == deal.buyer || msg.sender == deal.seller, "Not party");
        deal.status = Status.Disputed;
        emit DisputeRaised(dealId);
    }
}
