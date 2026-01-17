# CarbonCreditMarketplace Contract Documentation

## Overview

The `CarbonCreditMarketplace` contract is a marketplace for trading carbon credit NFTs using an ERC20-compatible payment token. It supports fixed-price listings and simple auctions, with platform fee support and administrative controls.

- **NFT Standard:** Interacts with a minimal ERC721-like interface (`INFTLite`)
- **Payment Token:** Any ERC20-compatible token (`IERC20Lite`)
- **Fee Recipient:** Platform vault address
- **Admin Controls:** Owner can pause, set fees, change vault, payment token, or NFT contract

## Features

- List NFTs for fixed price
- Buy NFTs with ERC20 token
- Start and participate in simple auctions
- Platform fee (basis points)
- Pausable by owner
- Admin can change key parameters

## Key Data Structures

- **Listing:**
  - `seller`: NFT owner
  - `price`: Sale price
  - `active`: Listing status
- **Auction:**
  - `seller`: NFT owner
  - `startPrice`: Minimum bid
  - `endAt`: Auction end timestamp
  - `highestBidder`: Current highest bidder
  - `highestBid`: Current highest bid
  - `active`: Auction status
  - `settled`: Auction settlement status

## Events

- `OwnerChanged(address oldOwner, address newOwner)`
- `Paused(bool v)`
- `Listed(uint256 tokenId, address seller, uint256 price)`
- `ListingCancelled(uint256 tokenId)`
- `Purchased(uint256 tokenId, address seller, address buyer, uint256 price)`
- `AuctionStarted(uint256 tokenId, address seller, uint256 startPrice, uint256 endAt)`
- `BidPlaced(uint256 tokenId, address bidder, uint256 amount)`
- `AuctionFinalized(uint256 tokenId, address seller, address winner, uint256 amount)`

## Functions

### Admin Functions
- `setOwner(address newOwner)` — Transfer contract ownership
- `setPaused(bool v)` — Pause/unpause contract
- `setFeeBps(uint16 bps)` — Set platform fee (max 20%)
- `setVault(address v)` — Set fee recipient
- `setPaymentToken(address t)` — Set ERC20 payment token
- `setNFT(address n)` — Set NFT contract

### Marketplace Functions
- `list(uint256 tokenId, uint256 price)` — List NFT for sale
- `cancelListing(uint256 tokenId)` — Cancel NFT listing
- `buy(uint256 tokenId)` — Buy listed NFT

### Auction Functions
- `startAuction(uint256 tokenId, uint256 startPrice, uint256 durationSeconds)` — Start auction
- `bid(uint256 tokenId, uint256 amount)` — Place bid (no escrow)
- `finalizeAuction(uint256 tokenId)` — Finalize auction, transfer NFT and funds

## Security & Limitations
- Only owner can change admin parameters
- Pausable for emergency stops
- No ETH support (ERC20 only)
- No bid escrow; highest bidder must pay at finalize
- No refunds for outbid bidders (minimal auction)

## Usage Example
1. Owner sets up contract with payment token, NFT, and vault
2. User lists NFT for sale
3. Buyer purchases NFT with ERC20 token
4. User starts auction for NFT
5. Bidders place bids (must approve token for each bid)
6. After auction ends, anyone can finalize and transfer NFT/funds

---

For ABI and integration details, see the contract source and generated artifacts.
