
## QUIZ QUESTION
Let’s say that each item in your RecyclerView list contains four individual data views, and you don't cache these views in a ViewHolder. If eight items fit on screen, approximately how many extra findViewById() calls will be made if you scroll through 30 items?

In addition to the eight items that fit on screen, assume that two extra items are needed for smooth scrolling.



## Thanks for completing that!
With only eight items on screen, we would need at least eight item views, but the question says that we need two extras for smooth scrolling, which means 10 items total. 10 items times 4 individual data views per item means 40 calls to FindViewById. And we could cache these views in a ViewHolder to fill our RecyclerView and then access them later when we scroll and recycle views. If we didn’t use a ViewHolder, we would have to call FindViewById 4 times for each of the 30 items we scroll through, that’s 30x4 or 120 calls to FindViewById.

So the difference between using a ViewHolder and not using one is 120-40 calls to FindViewById. 80 extra calls for not using a ViewHolder! Android phones today are so fast, that you probably wouldn't notice this optimization, but it will give you slightly better battery usage, which could be noticeable if you are scrolling through very large lists. So it’s best to use a ViewHolder.


