# CarbonCreditMarketplace Contract API

## Contract: CarbonCreditMarketplace

### Constructor
- `constructor(address paymentToken_, address nft_, address vault_)`
  - Sets the payment token, NFT contract, and fee vault.

### Admin Functions
- `setOwner(address newOwner)`
- `setPaused(bool v)`
- `setFeeBps(uint16 bps)`
- `setVault(address v)`
- `setPaymentToken(address t)`
- `setNFT(address n)`

### Marketplace Functions
- `list(uint256 tokenId, uint256 price)`
- `cancelListing(uint256 tokenId)`
- `buy(uint256 tokenId)`

### Auction Functions
- `startAuction(uint256 tokenId, uint256 startPrice, uint256 durationSeconds)`
- `bid(uint256 tokenId, uint256 amount)`
- `finalizeAuction(uint256 tokenId)`

### Events
- `OwnerChanged(address oldOwner, address newOwner)`
- `Paused(bool v)`
- `Listed(uint256 tokenId, address seller, uint256 price)`
- `ListingCancelled(uint256 tokenId)`
- `Purchased(uint256 tokenId, address seller, address buyer, uint256 price)`
- `AuctionStarted(uint256 tokenId, address seller, uint256 startPrice, uint256 endAt)`
- `BidPlaced(uint256 tokenId, address bidder, uint256 amount)`
- `AuctionFinalized(uint256 tokenId, address seller, address winner, uint256 amount)`

### Data Structures
- `struct Listing { address seller; uint256 price; bool active; }`
- `struct Auction { address seller; uint256 startPrice; uint256 endAt; address highestBidder; uint256 highestBid; bool active; bool settled; }`

### Notes
- All payments are in the configured ERC20 token.
- Platform fee is deducted on each sale/auction.
- No ETH support, no bid escrow, no refunds for outbid bidders.

---

For detailed usage, see the main contract documentation.
