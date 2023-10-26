![](my_logo.png)

# High Risk Findings (3)

- ## [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) function does not verify if sender is owner of the contract

  - The [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) function does not check to verify if the sender is in fact the owner of the contract
  
- ### Impact

  - This vulnerability allows anyone to set the password, and this is not the intended behaviour of the function
  
- ### Recommended Mitigation

  - In [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) function, add a `require()` function:

      ```javascript
       function setPassword(string memory newPassword) external {
           require(msg.sender == s_owner, "This function can only be called by the owner");
           s_password = newPassword;
       }
      ```
  
      The [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) function now checks if the sender is the owner of the contract before changing the password. This is done using the `require()` function, which throws an error if the condition is not met

- ## Improper implementation of the [`getPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) function

  - The function is supposed to return the password, but instead, it reverts with the `PasswordStore__NotOwner` error if the sender is not the owner
  
- ### Impact

  - This means that the password cannot be retrieved by the owner either
  
- ### Recommended Mitigation

    ```javascript
     function getPassword() external view returns (string memory) {
         require(msg.sender == s_owner, "This function can only be called by the owner");
         return s_password;
     }
    ```

    The [`getPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) function now returns the password if the sender is the owner. If the sender is not the owner, it returns the error statement

- ## Rewrite the whole contract 😐

  - From lack of modifiers, to improper checks done in the [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) and [`getPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) functions, to excessive gas usage, to not using gas saving tips, etc. This contract is a goldmine of bugs. Good news, bug slayer is here 🐜⚔

- ### Impact

  - A whole lotta bugs. We need a whole lotta frogs 🐸 \*croak
  
- ### Recommended Mitigation

  - I rewrote the [`PasswordStore.sol`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) contract, what are friends for 😎

      ```javascript
      // SPDX-License-Identifier: MIT
      pragma solidity 0.8.18;

      /*
       * @author not-so-secure-dev
       * @title PasswordStore
       * @notice This contract allows you to store a private password that others won't be able to see.
       * You can update your password at any time.
       */
      contract PasswordStore {
          error PasswordStore__NotOwner();

          address private immutable s_owner;
          string private s_password;

          event SetPassword(address indexed _by, string _password);

          constructor() {
              s_owner = msg.sender;
          }

          modifier onlyOwner() {
              if(msg.sender != s_owner) {
                  revert PasswordStore__NotOwner();
              }
              _;
          }

          /*
           * @notice This function allows only the owner to set a new password.
           * @param newPassword The new password to set.
           */
          function setPassword(string memory newPassword) external onlyOwner {
              s_password = newPassword;
              emit SetPassword(msg.sender, newPassword);
          }

          /*
           * @notice This allows only the owner to retrieve the password.
           * @return The current password.
           */
          function getPassword() external view onlyOwner returns (string memory) {
              return s_password;
          }
      }
      ```

# Medium Risk Findings (4)

- ## Forge not properly initialized

  - Forge is not properly initialized, thus **lib** folder is missing. This means **forge-std/Script.sol** and **forge-std/Test.sol** cannot be imported in the Script and Test files respectively
  
- ### Impact

  - Because **forge-std** is missing, deployments and tests cannot be done
  
- ### Recommended Mitigation

  - Run
  
      ```javascript
      forge init --force .
      ```

      to initialize the project properly (because it is not an empty directory). After initializing the project, delete the **Counter.sol**, **Counter.s.sol**, and **Counter.t.sol** files from the **src**, **script**, and **test** folders respectively.
    - I added a [screenshot](https://drive.google.com/file/d/19M6HOJWAbSCQdaQ6wk2ag7BIBhhOzJqe/view) of my terminal showing this finding
  
- ## No event emitted upon password change

  - The [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) function does not emit any event upon password change
  
- ### Impact

  - This makes it difficult to track changes to the password
  
- ### Recommended Mitigation

  - Correctly specify the event:
  
      ```javascript
      // event correctly specified
      event SetPassword(address indexed _by, string _password);
      ```

  - Properly implement the event in the [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) function:
  
        ```javascript
      // event properly implemented in setPassword function
      function setPassword(string memory newPassword) external {
              require(msg.sender == s_owner, "This function can only be called by the owner");
              s_password = newPassword;
              emit SetPassword(msg.sender, newPassword);
          }
      ```

# Gas Saving Tips (2)

- ## Declare `s_owner` variable as `immutable`

  - Currently, `s_owner` (which is the owner's address) is stored in storage
  
- ### Impact
  
  - Extra gas is used to retrieve `s_owner` from storage everytime this contract is called
  
- ### Recommended Mitigation
  
  - The `s_owner` variable should be declared as `immutable`. This means that its value can only be set once during the contract's construction and cannot be changed afterwards. This saves gas because the contract doesn't need to store the owner's address in storage, it can just use the msg.sender value from the constructor
  
        ```javascript
        address private immutable s_owner;
        ```
  
- ## Use a modifier to save gas

  - Currently, a `require()` function is used to restrict access to the [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) and [`getPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) functions to only the owner of the contract
  
- ### Impact
  
  - A check is done any time either of [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) and [`getPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) functions are called, and these checks would use extra gas
  
- ### Recommended Mitigation
  
  - A `onlyOwner` modifier should be created and used to restrict access to the [`setPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) and [`getPassword()`](https://github.com/Cyfrin/2023-10-PasswordStore/blob/main/src/PasswordStore.sol) functions. This modifier checks if the sender is the owner of the contract and throws an error if they are not. This saves gas because the check is done only once per function call, instead of being done in each function.
  
        ```javascript
        // create modifier
        modifier onlyOwner() {
                if(msg.sender != s_owner) {
                    revert PasswordStore__NotOwner();
                }
                _;
            }
        ```

        ```javascript
        // implement modifier in setPassword function
        function setPassword(string memory newPassword) external onlyOwner {
                s_password = newPassword;
                emit SetPassword(msg.sender, newPassword);
            }
        ```

        ```javascript
        // implement modifier in getPassword function
         function getPassword() external view onlyOwner returns (string memory) {
                return s_password;
            }
        ```
  