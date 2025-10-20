***4 sum***
- quadruplet is exsit for sure
- same element can't repeat in the result list

the brute force approach for this question would be, pick every four elements in this array to form a quadruplet. That would be the quatric level for the time complexity, for sure we don't want that

to make it better, i think we can split the quadruplet into pairs and match them with each other. That would be O(n^2) for create the pairs and O((n/2)^2) to match them, and extra spaces for creating the pairs and hashmap

I think we can separate a sub problem from this question, which is sum + nums[i] != target

base case:
sum + nums[i] == target, add the quadruplet in to the list
sum + nums[i] > target, return
sum + nums[i] < target, more
if len[temp] > 4, return



