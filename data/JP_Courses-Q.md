0. QA: The dev comment needs some clarity added, as per below.

https://github.com/code-423n4/2023-08-shell/blob/8ed551004f470489e070c1dd617d67eb4bf114e6/src/proteus/EvolvingProteus.sol#L38

The following should be added to the current comment on L38:

"...will result in transaction failure if the transaction amount is too small relative to the size of the reserves in the pool."


1. QA: The following should be added to the dev comment for clarity and completion, to eliminate any chance of misunderstandings/confusion:

https://github.com/code-423n4/2023-08-shell/blob/8ed551004f470489e070c1dd617d67eb4bf114e6/src/proteus/EvolvingProteus.sol#L39

L39:
"A transaction amount either as an input into the pool or an output from the pool will result in a transaction failure if the transaction amount is too large relative to the size of the reserves in the pool."


2. QA: The dev comment is partially correct, and the correction is below:

https://github.com/code-423n4/2023-08-shell/blob/8ed551004f470489e070c1dd617d67eb4bf114e6/src/proteus/EvolvingProteus.sol#L40

L40:
Replace the existing relevant part with this, because there's no `above` for this scenario/case:
"...in the pool going below this ratio will fail."


3. QA: Incorrect dev comment.

https://github.com/code-423n4/2023-08-shell/blob/8ed551004f470489e070c1dd617d67eb4bf114e6/src/proteus/EvolvingProteus.sol#L120

Comment is incorrect, should be: "the sq. root of x instantaneous price", as this is not inverse like for `a`.


4. QA: Incorrect dev comment.

https://github.com/code-423n4/2023-08-shell/blob/8ed551004f470489e070c1dd617d67eb4bf114e6/src/proteus/EvolvingProteus.sol#L240

corrected comment is: "px_final The final price at the x axis"


5. LOW: Input validation: no checks for whether px_init == px_final, and same for py...checks

https://github.com/code-423n4/2023-08-shell/blob/8ed551004f470489e070c1dd617d67eb4bf114e6/src/proteus/EvolvingProteus.sol#L249

