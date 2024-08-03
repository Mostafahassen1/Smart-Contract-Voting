# Project Idea: Decentralized Voting System

The decentralized voting system allows users to vote on proposals in a transparent and tamper-proof manner. Each proposal can have a voting period, and users can cast their votes during this period. Once the voting period ends, the contract will tally the votes and determine the outcome.

## Smart Contract Code (Solidity)

Here is a simple implementation of the decentralized voting system in Solidity:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Voting {
    struct Proposal {
        string description;
        uint256 voteCountYes;
        uint256 voteCountNo;
        uint256 votingDeadline;
        bool exists;
    }

    mapping(uint256 => Proposal) public proposals;
    uint256 public proposalCount;

    mapping(address => mapping(uint256 => bool)) public hasVoted;

    event ProposalCreated(uint256 indexed proposalId, string description, uint256 votingDeadline);
    event VoteCast(uint256 indexed proposalId, address indexed voter, bool vote);
    event VotingEnded(uint256 indexed proposalId, bool result);

    modifier proposalExists(uint256 _proposalId) {
        require(proposals[_proposalId].exists, "Proposal does not exist");
        _;
    }

    modifier hasNotVoted(uint256 _proposalId) {
        require(!hasVoted[msg.sender][_proposalId], "You have already voted");
        _;
    }

    modifier withinVotingPeriod(uint256 _proposalId) {
        require(block.timestamp < proposals[_proposalId].votingDeadline, "Voting period has ended");
        _;
    }

    function createProposal(string memory _description, uint256 _votingPeriod) external {
        uint256 proposalId = proposalCount++;
        proposals[proposalId] = Proposal({
            description: _description,
            voteCountYes: 0,
            voteCountNo: 0,
            votingDeadline: block.timestamp + _votingPeriod,
            exists: true
        });

        emit ProposalCreated(proposalId, _description, block.timestamp + _votingPeriod);
    }

    function vote(uint256 _proposalId, bool _vote) external proposalExists(_proposalId) hasNotVoted(_proposalId) withinVotingPeriod(_proposalId) {
        Proposal storage proposal = proposals[_proposalId];
        hasVoted[msg.sender][_proposalId] = true;

        if (_vote) {
            proposal.voteCountYes++;
        } else {
            proposal.voteCountNo++;
        }

        emit VoteCast(_proposalId, msg.sender, _vote);
    }

    function endVoting(uint256 _proposalId) external proposalExists(_proposalId) {
        Proposal storage proposal = proposals[_proposalId];
        require(block.timestamp >= proposal.votingDeadline, "Voting period has not ended yet");

        bool result = proposal.voteCountYes > proposal.voteCountNo;
        emit VotingEnded(_proposalId, result);
    }

    function getProposal(uint256 _proposalId) external view proposalExists(_proposalId) returns (string memory description, uint256 voteCountYes, uint256 voteCountNo, uint256 votingDeadline) {
        Proposal storage proposal = proposals[_proposalId];
        return (proposal.description, proposal.voteCountYes, proposal.voteCountNo, proposal.votingDeadline);
    }
}

```

# Explanation

## Structs

### Proposal
- **Description**: Holds the details of a proposal, including its description, vote counts, voting deadline, and existence.

## Mappings

### proposals
- **Description**: Maps a proposal ID to a Proposal struct.

### hasVoted
- **Description**: Tracks whether an address has voted on a specific proposal.

## Modifiers

### proposalExists
- **Description**: Ensures the proposal exists.

### hasNotVoted
- **Description**: Ensures the caller has not already voted on the proposal.

### withinVotingPeriod
- **Description**: Ensures the voting period has not ended.

## Functions

### createProposal
- **Description**: Creates a new proposal with a description and voting period.

### vote
- **Description**: Allows users to cast their votes on a proposal.

### endVoting
- **Description**: Ends the voting period and determines the result.

### getProposal
- **Description**: Retrieves the details of a proposal.
