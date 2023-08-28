# Approach
The approach i had taken for reviewing the Shell contest was first going through the documentation and mainly through the [Proteus: AMM Engine](https://wiki.shellprotocol.io/how-shell-works/proteus-amm-engine) in order to understand the operational fundamentals of the project. 
Then i started going through the code base and first went through the ABDKMath64x64 library to understand how it functions. 
Afterwards i go through all the function in a chronological order to find any discrepancy. The structural approach help me in getting issues in gas, low and medium severity classes.
In area of findings high or medium risk issues, i started going through codebase more deeply and going through every comment to find any discrepancy which results in successfully finding one medium issue.

# Learning
My learning from this project is to first understand the code base structure by going through the documents and then look for the codebase to find and discrepancy that may result in effecting the structural flow of project which may possibly lead to loss of funds for both users and project.

# Comments
My finding for this project is related to loss of accounting precision due to multiple divisions carried out before multiplication that increases the overall precision loss ratio.


### Time spent:
10 hours