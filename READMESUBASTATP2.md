# ethtrabajopractico
This final project - Module 2 of the Ethereum course implements a smart contract for an auction on the Ethereum blockchain using Solidity.
It allows participants to bid by sending ETH, and the winner is the one who offers the highest amount when the established time ends.
### Key Variables
*  owner; Address of the auction owner
*  startingPrice;Initial price of the auction
*  highestBid; Current highest bid
*  highestBidder;Address of the highest bidder
*  ended;Indicates if the auction has ended
*  endTime;End time of the auction
*  extensionTime = 10 minutes;Extension time of the auction (10 minutes)
*  COMMISSION_RATE = 2;Commission rate (2%)

Deploy and Test (Example with Remix)

1.  Open in Remix IDE and compile it with a compatible version of Solidity (e.g. `^0.8.20`).
2.  **Deploy:**
    *   Go to the tab 'Deploy & Run Transactions'.
    *   Select the environment (Example: "Remix VM" for quick tests, or "Injected Provider - MetaMask" for test networks like Sepolia).
    *   Place arguments for the constructor:
           -  uint _startingPrice
           -  uint _duration

This project is under the MIT license. See the `LICENSE` file (or the SPDX identifier in the code) for more details.

pragma solidity ^0.8.20;
// Auction contract with partial refund
contract Auction {
    // Address of the auction owner
    address public owner;

    // Initial price of the auction
    uint public startingPrice;

    // Current highest bid
    uint public highestBid;

    // Address of the highest bidder
    address public highestBidder;

    // Indicates if the auction has ended
    bool public ended;

    // End time of the auction
    uint public endTime;

    // Extension time of the auction (10 minutes)
    uint public extensionTime = 10 minutes;

    // Commission rate (2%)
    uint public constant COMMISSION_RATE = 2;

    // Struct to store bids
    struct Bid {
        // Address of the bidder
        address bidder;
        // Amount of the bid
        uint amount;
    }
    // Array of bids
    Bid[] public bids;

    // Mapping to store the amounts bid by each bidder
    mapping(address => uint) public bidderAmounts;

    // Events to notify new bids, auction end, and refunds
    event NewBid(address bidder, uint bid);// 
    event AuctionEnded(address winner, uint amount);
    event Refund(address bidder, uint amount);
    event LogBidder(address bidder, uint amount);
    event WithdrawExcess(address participant, uint amount);

    // Constructor to initialize the auction
    constructor(uint _startingPrice, uint _duration) {
        // Set the owner of the auction
        owner = msg.sender;
        // Set the starting price of the auction
        startingPrice = _startingPrice;
        // Set the highest bid to the starting price
        highestBid = _startingPrice;
        // Set the end time of the auction
        endTime = block.timestamp + _duration;
    }
    // Function to place a bid
    function bid(uint _bid) public {
        // Check if the auction has not ended
        require(!ended, "The auction has ended");
        // Check if the bid is at least 5% higher than the current bid
        require(_bid >= highestBid * 105 / 100, "The bid is insufficient. Must be at least 5% higher than the current bid");
        // Check if the end time needs to be extended
        if (block.timestamp >= endTime - 10 minutes) {
            // Extend the end time by 10 minutes
            endTime += extensionTime;
        }
        // Update the highest bid and bidder
        highestBid = _bid;
        highestBidder = msg.sender;
        bidderAmounts[msg.sender] = _bid;
        // Add the bid to the array of bids
        bids.push(Bid(msg.sender, _bid));
        // Emit a NewBid event
        emit NewBid(msg.sender, _bid);
    }

    // Function to end the auction
    function endAuction() public {
        // Check if the owner is the one ending the auction
        require(msg.sender == owner, "Only the owner can end the auction");
        // Check if the auction has ended
        require(block.timestamp >= endTime, "The auction has not ended yet");
        // Check if the auction has not already ended
        require(!ended, "The auction has already ended");
        // Mark the auction as ended
        ended = true;
        // Transfer the highest bid amount to the winner
        payable(highestBidder).transfer(highestBid);
        // Emit an AuctionEnded event
        emit AuctionEnded(highestBidder, highestBid);
    }
    // Function to refund bidders
    function refundBidders() public {
        // Check if the auction has ended
        require(ended, "The auction has not ended yet");
        // Iterate over the bids and refund bidders who did not win
        for (uint i = 0; i < bids.length; i++) {
            if (bids[i].bidder != highestBidder) {
                // Calculate the refund amount (bid amount minus commission)
                uint refundAmount = bids[i].amount * (100 - COMMISSION_RATE) / 100;
                // Transfer the refund amount to the bidder
                payable(bids[i].bidder).transfer(refundAmount);
                // Emit a Refund event
                emit Refund(bids[i].bidder, refundAmount);
            }
        }
    }
    // Function to get the list of bidders and their bid amounts
    function getBidderList() public view returns (address[] memory, uint[] memory) {
        // Create arrays to store the bidders and their bid amounts
        address[] memory bidders = new address[](bids.length);
        uint[] memory amounts = new uint[](bids.length);
        // Iterate over the bids and store the bidders and their bid amounts
        for (uint i = 0; i < bids.length; i++) {
            bidders[i] = bids[i].bidder;
            amounts[i] = bids[i].amount;
        }        return (bidders, amounts);    }
        }

