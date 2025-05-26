// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title Decentralized Autonomous Organization (DAO)
 * @dev A simple DAO implementation with proposal creation, voting, and execution
 * @author DAO Development Team
 */
contract DAO {
    // Struct to represent a proposal
    struct Proposal {
        uint256 id;
        string description;
        uint256 votesFor;
        uint256 votesAgainst;
        uint256 deadline;
        bool executed;
        address proposer;
        mapping(address => bool) hasVoted;
    }

    // State variables
    address public owner;
    uint256 public proposalCount;
    uint256 public votingDuration = 7 days;
    uint256 public minimumTokensToPropose = 100 * 10**18; // 100 tokens
    
    // Mappings
    mapping(uint256 => Proposal) public proposals;
    mapping(address => uint256) public tokenBalance;
    mapping(address => bool) public members;
    
    // Events
    event ProposalCreated(uint256 indexed proposalId, string description, address indexed proposer);
    event VoteCasted(uint256 indexed proposalId, address indexed voter, bool support, uint256 weight);
    event ProposalExecuted(uint256 indexed proposalId, bool success);
    event MemberAdded(address indexed member);
    event TokensDeposited(address indexed member, uint256 amount);

    // Modifiers
    modifier onlyMember() {
        require(members[msg.sender], "Only DAO members can perform this action");
        _;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can perform this action");
        _;
    }

    modifier validProposal(uint256 _proposalId) {
        require(_proposalId > 0 && _proposalId <= proposalCount, "Invalid proposal ID");
        _;
    }

    /**
     * @dev Constructor to initialize the DAO
     */
    constructor() {
        owner = msg.sender;
        members[msg.sender] = true;
        tokenBalance[msg.sender] = 1000 * 10**18; // Give owner initial tokens
        emit MemberAdded(msg.sender);
    }

    /**
     * @dev Core Function 1: Create a new proposal
     * @param _description Description of the proposal
     */
    function createProposal(string memory _description) external onlyMember {
        require(tokenBalance[msg.sender] >= minimumTokensToPropose, "Insufficient tokens to create proposal");
        require(bytes(_description).length > 0, "Proposal description cannot be empty");

        proposalCount++;
        
        Proposal storage newProposal = proposals[proposalCount];
        newProposal.id = proposalCount;
        newProposal.description = _description;
        newProposal.deadline = block.timestamp + votingDuration;
        newProposal.proposer = msg.sender;
        newProposal.executed = false;

        emit ProposalCreated(proposalCount, _description, msg.sender);
    }

    /**
     * @dev Core Function 2: Vote on a proposal
     * @param _proposalId ID of the proposal to vote on
     * @param _support True for yes, false for no
     */
    function vote(uint256 _proposalId, bool _support) external onlyMember validProposal(_proposalId) {
        Proposal storage proposal = proposals[_proposalId];
        
        require(block.timestamp < proposal.deadline, "Voting period has ended");
        require(!proposal.hasVoted[msg.sender], "You have already voted on this proposal");
        require(tokenBalance[msg.sender] > 0, "You need tokens to vote");

        proposal.hasVoted[msg.sender] = true;
        uint256 voterWeight = tokenBalance[msg.sender];

        if (_support) {
            proposal.votesFor += voterWeight;
        } else {
            proposal.votesAgainst += voterWeight;
        }

        emit VoteCasted(_proposalId, msg.sender, _support, voterWeight);
    }

    /**
     * @dev Core Function 3: Execute a proposal after voting period
     * @param _proposalId ID of the proposal to execute
     */
    function executeProposal(uint256 _proposalId) external validProposal(_proposalId) {
        Proposal storage proposal = proposals[_proposalId];
        
        require(block.timestamp >= proposal.deadline, "Voting period is still active");
        require(!proposal.executed, "Proposal has already been executed");
        require(proposal.votesFor > proposal.votesAgainst, "Proposal did not pass");

        proposal.executed = true;
        
        // Here you would implement the actual execution logic
        // For this example, we'll just emit an event
        bool success = true; // Placeholder for actual execution result
        
        emit ProposalExecuted(_proposalId, success);
    }

    /**
     * @dev Add a new member to the DAO (only owner)
     * @param _newMember Address of the new member
     */
    function addMember(address _newMember) external onlyOwner {
        require(_newMember != address(0), "Invalid address");
        require(!members[_newMember], "Address is already a member");
        
        members[_newMember] = true;
        tokenBalance[_newMember] = 50 * 10**18; // Give new members some initial tokens
        
        emit MemberAdded(_newMember);
        emit TokensDeposited(_newMember, 50 * 10**18);
    }

    /**
     * @dev Deposit tokens to increase voting power
     */
    function depositTokens() external payable onlyMember {
        require(msg.value > 0, "Must deposit some ETH");
        
        // Convert ETH to tokens (1 ETH = 1000 tokens for simplicity)
        uint256 tokensToAdd = msg.value * 1000;
        tokenBalance[msg.sender] += tokensToAdd;
        
        emit TokensDeposited(msg.sender, tokensToAdd);
    }

    /**
     * @dev Get proposal details
     * @param _proposalId ID of the proposal
     * @return id, description, votesFor, votesAgainst, deadline, executed, proposer
     */
    function getProposal(uint256 _proposalId) external view validProposal(_proposalId) 
        returns (uint256, string memory, uint256, uint256, uint256, bool, address) {
        Proposal storage proposal = proposals[_proposalId];
        return (
            proposal.id,
            proposal.description,
            proposal.votesFor,
            proposal.votesAgainst,
            proposal.deadline,
            proposal.executed,
            proposal.proposer
        );
    }

    /**
     * @dev Check if an address has voted on a specific proposal
     * @param _proposalId ID of the proposal
     * @param _voter Address of the voter
     * @return bool indicating if the address has voted
     */
    function hasVoted(uint256 _proposalId, address _voter) external view validProposal(_proposalId) returns (bool) {
        return proposals[_proposalId].hasVoted[_voter];
    }

    /**
     * @dev Get member's token balance
     * @param _member Address of the member
     * @return Token balance of the member
     */
    function getMemberTokenBalance(address _member) external view returns (uint256) {
        return tokenBalance[_member];
    }

    /**
     * @dev Check if address is a DAO member
     * @param _address Address to check
     * @return bool indicating membership status
     */
    function isMember(address _address) external view returns (bool) {
        return members[_address];
    }

    /**
     * @dev Get total number of proposals
     * @return Total proposal count
     */
    function getTotalProposals() external view returns (uint256) {
        return proposalCount;
    }

    /**
     * @dev Withdraw contract balance (only owner)
     */
    function withdraw() external onlyOwner {
        uint256 balance = address(this).balance;
        require(balance > 0, "No funds to withdraw");
        
        (bool success, ) = payable(owner).call{value: balance}("");
        require(success, "Withdrawal failed");
    }

    /**
     * @dev Fallback function to receive ETH
     */
    receive() external payable {
        // Contract can receive ETH
    }
}
