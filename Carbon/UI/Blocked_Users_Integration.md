# Blocked Users Integration Guide

This guide shows how to integrate the blocked users functionality into various components of the carbon credit marketplace.

## 1. Core Files Added

- `frontend/src/context/BlockedUsersProvider.js` - Global state management for blocked users
- `frontend/src/components/BlockedUserDialog.js` - Reusable dialog for blocked user warnings
- `frontend/src/hooks/useBlockedUserCheck.js` - Hook for checking blocked users

## 2. App.js Integration

The `BlockedUsersProvider` has been added to `App.js` to wrap the entire application:

```javascript
import { BlockedUsersProvider } from "./context/BlockedUsersProvider";

return (
  <BlockedUsersProvider>
    <WebSocketProvider>
      {/* Rest of app */}
    </WebSocketProvider>
  </BlockedUsersProvider>
);
```

## 3. Component Integration Examples

### AdminUserAnalytics.js
- Uses the global blocked users context
- Persists blocked status across page reloads
- Updates user status based on blocked state

### MarketplaceItem.js
- Protects Buy, List, and Bid actions
- Shows blocked user dialog when action is blocked
- Gets user address before checking blocked status

## 4. Integration Template for Other Components

To add blocked user protection to any component:

```javascript
import { useBlockedUsers } from '../context/BlockedUsersProvider';
import BlockedUserDialog from './BlockedUserDialog';
import useBlockedUserCheck from '../hooks/useBlockedUserCheck';
import { BrowserProvider } from 'ethers';

export default function YourComponent() {
  const { checkUserBlocked, blockedDialogOpen, blockedUserAddress, blockedAction, closeBlockedDialog } = useBlockedUserCheck();

  const handleProtectedAction = async (actionName) => {
    // Get current user address
    const provider = new BrowserProvider(window.ethereum);
    const accounts = await provider.listAccounts();
    const userAddress = accounts[0]?.address;
    
    // Check if user is blocked
    if (checkUserBlocked(userAddress, actionName)) {
      return; // Block the action - dialog will show automatically
    }
    
    // Proceed with action
    // ... your action code here
  };

  return (
    <>
      {/* Your component JSX */}
      <button onClick={() => handleProtectedAction('mint NFTs')}>
        Mint NFT
      </button>
      
      {/* Add the blocked user dialog */}
      <BlockedUserDialog
        open={blockedDialogOpen}
        onClose={closeBlockedDialog}
        userAddress={blockedUserAddress}
        action={blockedAction}
        title="Action Restricted"
      />
    </>
  );
}
```

## 5. Actions to Protect

The following marketplace actions should be protected with blocked user checks:

1. **Buy NFTs** - MarketplaceItem.js ✅
2. **List NFTs for sale** - MarketplaceItem.js ✅
3. **Place bids on auctions** - MarketplaceItem.js ✅
4. **Mint new NFTs** - CarbonDashboard.js (needs integration)
5. **Retire NFTs** - CarbonDashboard.js (needs integration)
6. **Cancel listings** - MarketplaceItem.js (needs integration)
7. **Create auctions** - Any auction creation component (needs integration)

## 6. Features

### Persistence
- Blocked users can be stored in localStorage (basic) or IPFS (recommended)
- IPFS storage provides decentralized, persistent, cross-device synchronization
- Status persists across browser sessions, devices, and admin accounts

### Global State
- Blocked status is shared across all components
- Changes in admin panel immediately affect marketplace actions

### User Experience
- Clear dialog explaining why action is blocked
- Consistent messaging across all protected actions
- Copy-to-clipboard functionality for support requests

### Admin Features
- Toggle block/unblock with visual feedback
- Filter users by blocked status
- Export includes blocked status

## 7. Storage Options

### Current: localStorage (Basic)
- Stored in browser localStorage with key `'blockedUsers'`
- Device-specific, not shared across admin accounts
- Can be cleared by user

### Recommended: IPFS Storage (Since you have IPFS server)
Since you already host your own IPFS server for NFT metadata, you can use it for blocked user storage:

```javascript
// Enhanced BlockedUsersProvider with IPFS support
export const BlockedUsersProvider = ({ children }) => {
  const [blockedUsers, setBlockedUsers] = useState(new Set());
  const [ipfsHash, setIpfsHash] = useState(null);

  // Load from IPFS on startup
  useEffect(() => {
    loadBlockedUsersFromIPFS();
  }, []);

  const loadBlockedUsersFromIPFS = async () => {
    try {
      // Get current IPFS hash from localStorage or contract
      const currentHash = localStorage.getItem('blockedUsersHash');
      if (currentHash) {
        const response = await fetch(`${IPFS_GATEWAY_URL}/ipfs/${currentHash}`);
        const data = await response.json();
        
        if (data.version && data.blockedUsers) {
          setBlockedUsers(new Set(data.blockedUsers));
          setIpfsHash(currentHash);
        }
      }
    } catch (error) {
      console.warn('Failed to load from IPFS, falling back to localStorage');
      loadFromLocalStorage();
    }
  };

  const saveBlockedUsersToIPFS = async (updatedUsers) => {
    try {
      const blocklistData = {
        version: 1,
        timestamp: Date.now(),
        blockedUsers: Array.from(updatedUsers),
        adminSignature: await getAdminSignature() // Optional: sign with admin wallet
      };

      // Upload to your IPFS server
      const formData = new FormData();
      formData.append('file', new Blob([JSON.stringify(blocklistData)], {type: 'application/json'}));
      
      const response = await fetch(`${IPFS_SERVER_URL}/api/v0/add`, {
        method: 'POST',
        body: formData
      });
      
      const result = await response.json();
      const newHash = result.Hash;
      
      // Store hash for future reference
      localStorage.setItem('blockedUsersHash', newHash);
      setIpfsHash(newHash);
      
      // Optionally store hash on blockchain for decentralized reference
      await storeHashOnChain(newHash);
      
    } catch (error) {
      console.error('Failed to save to IPFS:', error);
      // Fallback to localStorage
      localStorage.setItem('blockedUsers', JSON.stringify(Array.from(updatedUsers)));
    }
  };
};
```

### IPFS Integration Benefits:
- **Cross-device sync**: All admin accounts see same blocked users
- **Persistent**: Cannot be cleared by users
- **Decentralized**: Stored on your IPFS infrastructure
- **Versioned**: Track changes over time
- **Signed**: Optional admin signature validation

### IPFS Implementation Steps:
1. Create blocked user list upload endpoint on your IPFS server
2. Store IPFS hash in smart contract or localStorage
3. Load blocked users from IPFS on app startup
4. Update IPFS when admins block/unblock users

## 8. Next Steps

To complete the integration:

1. Add blocked user checks to minting functionality
2. Add blocked user checks to retirement functionality  
3. Add blocked user checks to listing cancellation
4. Consider adding API endpoint for persistent server-side blocking
5. Add blockchain-based blocking (smart contract integration)
