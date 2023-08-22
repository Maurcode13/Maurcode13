- 👋 Hi, I’m @Maurcod
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
Maurcode13/Maurcode13 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
:

solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/HasSecondarySaleFees.sol";

contract CombinedToken is ERC20, Ownable, Pausable, ReentrancyGuard, HasSecondarySaleFees {
    address public custodian;
    address public minerRewardAddress;
    uint256 public miningReward;
    mapping(address => bool) private _whitelist;

    event GoldTokensMinted(address indexed recipient, uint256 amount);
    event MiningReward(address indexed miner, uint256 value);

    modifier onlyCustodian() {
        require(msg.sender == custodian, "Somente o custodiante pode chamar esta função");
        _;
    }

    modifier onlyMiner() {
        require(msg.sender == minerRewardAddress, "Apenas o minerador designado pode chamar isso");
        _;
    }

    constructor(address _custodian, address _minerRewardAddress) ERC20("CombinedToken", "CTK") {
        require(_custodian != address(0), "Endereço de custodiante inválido");
        require(_minerRewardAddress != address(0), "Endereço de recompensa do minerador inválido");

        custodian = _custodian;
        minerRewardAddress = _minerRewardAddress;
        miningReward = 50 * 10**decimals();
    }

    function mintGoldTokens(address recipient, uint256 amount) external onlyCustodian {
        require(recipient != address(0), "Endereço de destinatário inválido");
        require(amount > 0, "Quantidade inválida");
        
        _mint(recipient, amount);
        emit GoldTokensMinted(recipient, amount);
    }

    function redeemGoldTokens(uint256 amount) external {
        require(amount > 0, "Quantidade inválida");
        require(balanceOf(msg.sender) >= amount, "Saldo insuficiente");
        
        _burn(msg.sender, amount);
    }

    function mineBlock() external onlyMiner {
        uint256 currentBalance = balanceOf(minerRewardAddress);
        require(currentBalance + miningReward >= currentBalance, "Estouro de inteiro");
        
        _mint(minerRewardAddress, miningReward);
        emit MiningReward(minerRewardAddress, miningReward);
    }

    function changeMinerRewardAddress(address newMinerRewardAddress) external onlyMiner {
        require(newMinerRewardAddress != address(0), "Endereço de recompensa do minerador inválido");
        minerRewardAddress = newMinerRewardAddress;
    }

    function changeMiningReward(uint256 newMiningReward) external onlyMiner {
        miningReward = newMiningReward;
    }

    function addAddressToWhitelist(address account) external onlyOwner {
        _whitelist[account] = true;
    }

    function removeAddressFromWhitelist(address account) external onlyOwner {
        _whitelist[account] = false;
    }

    function isAddressWhitelisted(address account) external view returns (bool) {
        return _whitelist[account];
    }

    function _secondarySaleFees(uint256 tokenId) internal view override returns (address payable[] memory recipients, uint256[] memory fees) {
        recipients = new address payable[](1);
        fees = new uint256[](1);
        
        recipients[0] = payable(owner()); // O proprietário recebe uma taxa de venda sec
        fees[0] = 1000; // Define a taxa de venda sec como 10%
        
        return (recipients, fees);
    }
}
