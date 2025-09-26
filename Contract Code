// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title RightProof
 * @dev A smart contract for decentralized intellectual property and digital rights management
 * @author RightProof Team
 */
contract RightProof {
    
    // Struct to store intellectual property details
    struct IPAsset {
        uint256 id;
        address owner;
        string title;
        string description;
        string ipfsHash;
        uint256 timestamp;
        bool isActive;
        uint256 licensePrice;
    }
    
    // Struct to store license information
    struct License {
        uint256 assetId;
        address licensee;
        uint256 expirationDate;
        bool isActive;
        uint256 price;
    }
    
    // State variables
    mapping(uint256 => IPAsset) public ipAssets;
    mapping(uint256 => License[]) public assetLicenses;
    mapping(address => uint256[]) public ownerAssets;
    mapping(address => uint256[]) public licenseeAssets;
    
    uint256 public nextAssetId;
    uint256 public totalAssets;
    address public contractOwner;
    uint256 public platformFeePercentage = 5; // 5% platform fee
    
    // Events
    event AssetRegistered(
        uint256 indexed assetId, 
        address indexed owner, 
        string title, 
        uint256 timestamp
    );
    
    event LicensePurchased(
        uint256 indexed assetId, 
        address indexed licensee, 
        uint256 price, 
        uint256 expirationDate
    );
    
    event OwnershipTransferred(
        uint256 indexed assetId, 
        address indexed previousOwner, 
        address indexed newOwner
    );
    
    // Modifiers
    modifier onlyAssetOwner(uint256 _assetId) {
        require(ipAssets[_assetId].owner == msg.sender, "Not the asset owner");
        require(ipAssets[_assetId].isActive, "Asset is not active");
        _;
    }
    
    modifier onlyContractOwner() {
        require(msg.sender == contractOwner, "Not the contract owner");
        _;
    }
    
    modifier validAsset(uint256 _assetId) {
        require(_assetId < nextAssetId, "Asset does not exist");
        require(ipAssets[_assetId].isActive, "Asset is not active");
        _;
    }
    
    // Constructor
    constructor() {
        contractOwner = msg.sender;
        nextAssetId = 1;
    }
    
    /**
     * @dev Register a new intellectual property asset
     * @param _title Title of the IP asset
     * @param _description Description of the IP asset
     * @param _ipfsHash IPFS hash of the asset file
     * @param _licensePrice Price for licensing the asset (in wei)
     */
    function registerAsset(
        string memory _title,
        string memory _description,
        string memory _ipfsHash,
        uint256 _licensePrice
    ) external returns (uint256) {
        require(bytes(_title).length > 0, "Title cannot be empty");
        require(bytes(_ipfsHash).length > 0, "IPFS hash cannot be empty");
        require(_licensePrice > 0, "License price must be greater than 0");
        
        uint256 assetId = nextAssetId;
        
        ipAssets[assetId] = IPAsset({
            id: assetId,
            owner: msg.sender,
            title: _title,
            description: _description,
            ipfsHash: _ipfsHash,
            timestamp: block.timestamp,
            isActive: true,
            licensePrice: _licensePrice
        });
        
        ownerAssets[msg.sender].push(assetId);
        nextAssetId++;
        totalAssets++;
        
        emit AssetRegistered(assetId, msg.sender, _title, block.timestamp);
        
        return assetId;
    }
    
    /**
     * @dev Purchase a license for an IP asset
     * @param _assetId ID of the asset to license
     * @param _licenseDuration Duration of the license in seconds
     */
    function purchaseLicense(
        uint256 _assetId, 
        uint256 _licenseDuration
    ) external payable validAsset(_assetId) {
        IPAsset storage asset = ipAssets[_assetId];
        require(asset.owner != msg.sender, "Cannot license your own asset");
        require(msg.value >= asset.licensePrice, "Insufficient payment");
        require(_licenseDuration > 0, "License duration must be greater than 0");
        
        uint256 platformFee = (msg.value * platformFeePercentage) / 100;
        uint256 ownerPayment = msg.value - platformFee;
        
        // Transfer payment to asset owner
        payable(asset.owner).transfer(ownerPayment);
        
        // Create license record
        License memory newLicense = License({
            assetId: _assetId,
            licensee: msg.sender,
            expirationDate: block.timestamp + _licenseDuration,
            isActive: true,
            price: msg.value
        });
        
        assetLicenses[_assetId].push(newLicense);
        licenseeAssets[msg.sender].push(_assetId);
        
        emit LicensePurchased(
            _assetId, 
            msg.sender, 
            msg.value, 
            block.timestamp + _licenseDuration
        );
    }
    
    /**
     * @dev Transfer ownership of an IP asset
     * @param _assetId ID of the asset to transfer
     * @param _newOwner Address of the new owner
     */
    function transferOwnership(
        uint256 _assetId, 
        address _newOwner
    ) external onlyAssetOwner(_assetId) {
        require(_newOwner != address(0), "New owner cannot be zero address");
        require(_newOwner != msg.sender, "Cannot transfer to yourself");
        
        IPAsset storage asset = ipAssets[_assetId];
        address previousOwner = asset.owner;
        
        // Update asset owner
        asset.owner = _newOwner;
        
        // Update ownership mappings
        ownerAssets[_newOwner].push(_assetId);
        
        // Remove from previous owner's assets
        uint256[] storage prevOwnerAssets = ownerAssets[previousOwner];
        for (uint256 i = 0; i < prevOwnerAssets.length; i++) {
            if (prevOwnerAssets[i] == _assetId) {
                prevOwnerAssets[i] = prevOwnerAssets[prevOwnerAssets.length - 1];
                prevOwnerAssets.pop();
                break;
            }
        }
        
        emit OwnershipTransferred(_assetId, previousOwner, _newOwner);
    }
    
    // View functions
    
    /**
     * @dev Get asset details
     * @param _assetId ID of the asset
     */
    function getAsset(uint256 _assetId) external view validAsset(_assetId) returns (
        uint256 id,
        address owner,
        string memory title,
        string memory description,
        string memory ipfsHash,
        uint256 timestamp,
        bool isActive,
        uint256 licensePrice
    ) {
        IPAsset storage asset = ipAssets[_assetId];
        return (
            asset.id,
            asset.owner,
            asset.title,
            asset.description,
            asset.ipfsHash,
            asset.timestamp,
            asset.isActive,
            asset.licensePrice
        );
    }
    
    /**
     * @dev Get all assets owned by an address
     * @param _owner Address of the owner
     */
    function getOwnerAssets(address _owner) external view returns (uint256[] memory) {
        return ownerAssets[_owner];
    }
    
    /**
     * @dev Get all licenses for an asset
     * @param _assetId ID of the asset
     */
    function getAssetLicenses(uint256 _assetId) external view validAsset(_assetId) returns (License[] memory) {
        return assetLicenses[_assetId];
    }
    
    /**
     * @dev Check if a user has a valid license for an asset
     * @param _assetId ID of the asset
     * @param _licensee Address of the potential licensee
     */
    function hasValidLicense(uint256 _assetId, address _licensee) external view returns (bool) {
        License[] storage licenses = assetLicenses[_assetId];
        
        for (uint256 i = 0; i < licenses.length; i++) {
            if (licenses[i].licensee == _licensee && 
                licenses[i].isActive && 
                licenses[i].expirationDate > block.timestamp) {
                return true;
            }
        }
        
        return false;
    }
    
    // Admin functions
    
    /**
     * @dev Update platform fee (only contract owner)
     * @param _newFeePercentage New fee percentage
     */
    function updatePlatformFee(uint256 _newFeePercentage) external onlyContractOwner {
        require(_newFeePercentage <= 10, "Fee cannot exceed 10%");
        platformFeePercentage = _newFeePercentage;
    }
    
    /**
     * @dev Withdraw platform fees (only contract owner)
     */
    function withdrawPlatformFees() external onlyContractOwner {
        uint256 balance = address(this).balance;
        require(balance > 0, "No fees to withdraw");
        payable(contractOwner).transfer(balance);
    }
    
    /**
     * @dev Get contract balance
     */
    function getContractBalance() external view onlyContractOwner returns (uint256) {
        return address(this).balance;
    }
}
