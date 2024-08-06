# List FunC Library (`list.fc`)

The `list.fc` library provides a simple yet powerful implementation for managing serialized lists of arbitrary length in FunC. It includes essential primitives to load, process, and iterate over these lists, making it invaluable for smart-contract development on the TON blockchain.

[list.fc](./contracts/imports/list.fc)

## Introduction
Lists in FunC, commonly represented as tuples, can be cumbersome to handle when dealing with extended data structures. This library facilitates the storage and retrieval of lists in a structured way using FunC cells and slices.

## Primitives

### Loading a List
- **`load_list(slice body) impure`**: Loads the list from the reference in a larger object.

  **Parameters:**
    - `body`: A slice that contains a reference to the list.

  **Returns:**
    - The updated body slice, and the list slice.

  **Example Usage:**
  ```func
  (slice, slice) load_list(slice body)
  ```

### Checking if List Ended
- **`end(slice list) impure`**: Checks whether the list has ended.

  **Parameters:**
    - `list`: The slice which contains the list.

  **Returns:**
    - The updated list slice, and an integer which is `1` if the list has ended, else `0`.

  **Example Usage:**
  ```func
  (slice, int) end(slice list)
  ```

### Moving to the Next Chunk
- **`next(slice list) impure`**: Moves to the next chunk in the list.

  **Parameters:**
    - `list`: The slice which contains the current chunk of the list.

  **Returns:**
    - The updated list slice, and the next chunk slice.

  **Example Usage:**
  ```func
  (slice, slice) next(slice list)
  ```

## Iterating Over a List

Consider a smart contract that needs to iterate over a list of recipients and amounts to distribute some tokens. Here’s how you can iterate over an arbitrary long list in FunC:

### FunC Example
```func
#include "imports/list.fc";

() distribute_tokens(slice body) impure {

    ;; Load the list from the body
    (slice body, slice list) = load_list(body);

    ;; Initialize distribution
    while (true) {
        ;; Check if the list has ended
        (list, int ended) = end(list);
        if (ended) {
            ;; Exit the loop if the list has ended
            return ();
        }

        ;; Process current chunk
        slice recipient = list~load_msg_addr();
        int amount = list~load_coins();

        ;; Send tokens to recipient (example logic)
        send_tokens(recipient, amount);

        ;; Move to the next chunk
        (list, slice next_chunk) = next(list);
    }
}

() send_tokens(slice recipient, int amount) impure {
    ;; Example function to send tokens
    ;; Implement token sending logic here
}
```

### Building List Data in Blueprint TypeScript

To build the list data for the FunC contract in the Blueprint TypeScript platform, you can create a helper function that constructs a cell encapsulating the list.

```typescript
import { beginCell, Cell, Address } from '@ton/core';

/**
 * Builds a serialized list of recipients and amounts for FunC contract.
 * @param {Array<{recipient: Address, amount: bigint}>} items - List of items containing recipient addresses and token amounts.
 * @returns {Cell} Cell containing the serialized list.
 */
function buildList(items: Array<{ recipient: Address, amount: bigint }>): Cell {
    let rootCell: any = beginCell().endCell();
    for (let i = items.length - 1; i >= 0; i--) {
        const previousCell = rootCell;
        
        // Create a new rootCell
        rootCell = beginCell();

        // Store reference to the next cell
        rootCell.storeRef(previousCell);

        // Store recipient address and amount
        rootCell.storeAddress(items[i].recipient);
        rootCell.storeCoins(items[i].amount);

        rootCell = rootCell.endCell();
    }

    return rootCell;
}

// Example usage:
const recipients = [
    { recipient: new Address('EQD123...'), amount: BigInt(1000000000) }, // 1 TON
    { recipient: new Address('EQD456...'), amount: BigInt(1500000000) }  // 1.5 TON
];

const serializedListCell = buildList(recipients);
console.log(serializedListCell.toString()); // Serialized cell ready to be used in contract calls
```

The above TypeScript helper function constructs a serialized list compatible with the `list.fc` library, making it easy to integrate with FunC contracts expecting list-based data structures.

## Conclusion

The `list.fc` library is a valuable tool for handling serialized lists in FunC. By leveraging its primitives, developers can efficiently work with lists in smart contracts, ensuring robust and scalable solutions. The provided examples demonstrate how to iterate over lists in FunC and build compatible data in Blueprint TypeScript, bridging the gap between on-chain and off-chain logic.
