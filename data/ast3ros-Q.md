# [L-1] Consisten fee charge

In the current implementation, the fee is charged twice, each with approximately 0.125% so the total will be 0.25%. However, this implementation will lead to inconsistent charges for users since the fee is calculated before and after the operation.

0.125% fee before the operation, applied to the input

https://github.com/code-423n4/2023-08-shell/blob/514c3c34f7c0eef47c91d5aff7cf5e4b31fc56be/src/proteus/EvolvingProteus.sol#L516-L520

0.125% fee after the operation, applied to the output

https://github.com/code-423n4/2023-08-shell/blob/514c3c34f7c0eef47c91d5aff7cf5e4b31fc56be/src/proteus/EvolvingProteus.sol#L548

This is unfair for users. For example, with the same input amount to swap, two users A and B should be charged the same fee. However, this is not the case. For example: if user A waits for the right moment when the bonding curve evolves and gets the maximum number of output, they will be charged more fee than user B who swaps at the beginning of the bonding curve evolution and receives less.

## Recommended Mitigation Steps

Apply the fee to the input amount or the output amount only instead separating it into two parts.

# [L-2] The duration can be set to zero

In the constructor, the duration for which the curve evolves can be set to zero. This will cause the config.t() to always revert because of dividing by zero (duration becoming zero). It then makes the deposit, withdraw and swap functions unworkable.

https://github.com/code-423n4/2023-08-shell/blob/514c3c34f7c0eef47c91d5aff7cf5e4b31fc56be/src/proteus/EvolvingProteus.sol#L248

## Recommended Mitigation Steps

Check the duration and revert if it is zero in the constructor.

# [NC-1] Using brackets for code consistency in ternary operator

There is inconsistency in using brackets for ternary operator. In line 248, the brackets are used for the ternary operator, but in line 250, the brackets are not used. Using brackets for ternary operator is recommended for code consistency and readability.

Instance:
https://github.com/code-423n4/2023-08-shell/blob/514c3c34f7c0eef47c91d5aff7cf5e4b31fc56be/src/proteus/EvolvingProteus.sol#L301
https://github.com/code-423n4/2023-08-shell/blob/514c3c34f7c0eef47c91d5aff7cf5e4b31fc56be/src/proteus/EvolvingProteus.sol#L341

# [NC-2] Remove non exist case

In getting utility function, it checks if a < 0 and b < 0: 
        
        if(a < 0 && b < 0) utility = (r0 > r1) ? r1 : r0;

https://github.com/code-423n4/2023-08-shell/blob/514c3c34f7c0eef47c91d5aff7cf5e4b31fc56be/src/proteus/EvolvingProteus.sol#L720

However, by the calculation of a and b, a and b can never be less than 0:

        a = sqrt(1/(y_axis_price))
        b = sqrt(x_axis_price)

https://github.com/code-423n4/2023-08-shell/blob/514c3c34f7c0eef47c91d5aff7cf5e4b31fc56be/src/proteus/EvolvingProteus.sol#L111-L117
https://github.com/code-423n4/2023-08-shell/blob/514c3c34f7c0eef47c91d5aff7cf5e4b31fc56be/src/proteus/EvolvingProteus.sol#L119-L125

Therefore, it is better to remove the case of a < 0 and b < 0 for clearer code.