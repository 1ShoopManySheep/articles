# C++ - From goto to std::transform

In nearly every programming languages there are often many different possibilities to write code that in the end does exactly the same thing. In most cases we want to use the version that is the easiest to understand to a human reader.

Let's say we have a vector of integers and want another one with all elements being the square of the elements in the first one.
`[4, 1, 7] -> [8, 1, 49]`


## Goto
Sure, one could use gotos for this, but I guess nobody in their right mind would do this voluntarily, unless for trolling:

```c++
vector<int> squareVec1(const vector<int>& v)
{
    vector<int> result;
    result.reserve(v.size());
    auto it = begin(v);
    loopBegin:
    if (it == end(v))
        goto loopEnd;
    result.push_back(*it * *it);
    ++it;
    goto loopBegin;
    loopEnd:
    return result;
}
```
On the first look you have no idea what this code is doing. You have to kind of interpret it in your head and follow the control flow manually to find out what's going on.


## While loop
The next more readable step would be to use a loop as control structure.
```c++
vector<int> squareVec2(const vector<int>& v)
{
    vector<int> result;
    result.reserve(v.size());
    auto it = begin(v);
    while (it != end(v))
    {
        result.push_back(*it * *it);
        ++it;
    }
    return result;
}
```
This is bit better, at least you can immediatly see that there is some kind of loop, but most people would probably use a


## For loop
```c++
vector<int> squareVec3(const vector<int>& v)
{
    vector<int> result;
    result.reserve(v.size());
    for (auto it = begin(v); it != end(v); ++it)
    {
        result.push_back(*it * *it);
    }
    return result;
}
```
Here you can immediatly see that the algorithm iterates ofter the elements of `v` but you still have to read the whole line `for (auto it = begin(v); it != end(v); ++it)` until you know that every single element is used and not e.g. every second, since the increase could also be `it += 2` or something else instead of `++it`.


## Range-based for loop
```c++
vector<int> squareVec4(const vector<int>& v)
{
    vector<int> result;
    result.reserve(v.size());
    for (int i : v)
    {
        result.push_back(i*i);
    }
    return result;
}
```
This time the `for` line already tells the reader that probably every element of `v` is used, but still only probably. One still has to look into the body of the for loop and look for `if`, `continue` or even `break` statements to really know that `result` is guaranteed to have the same size as `v` in the end.

Many people stop here, but we can do better in terms of readability ease.


## std::accumulate
OK, how can we express more clearly without explicit comments what our code does, i.e. make it self explaining?
```c++
vector<int> squareVec5(const vector<int>& v)
{
    return accumulate(begin(v), end(v), vector<int>(),
                      [](vector<int> acc, int i)
    {
        acc.push_back(i*i);
        return acc;
    });
}
```
We use `std::accumulate`. Everybody reading this knows without thinking, that every element of `v` will be iterated over and that these values probably will be used to generate one resulting value. But apart from the performance problem of this solution, the "loop header" still does not say something about the shape of the result.


## std::transform
Eventually this is our final form.
```c++
vector<int> squareVec6(const vector<int>& v)
{
    vector<int> result;
    result.reserve(v.size());
    transform(begin(v), end(v), back_inserter(result), [](int i)
    {
        return i*i;
    });
    return result;
}
```
`std::transform` tells the reader at one glance that `result` will have the same size as `v` and that every single element from `v` will be used to generate exactly one element for `result`.
Now one just has to look at `return i*i` and he directly knows everything.
This is much easier than to decypher a for loop every time.

A for loop also beginning with `for (int i : v)` could do something totally unrelated to `std::transform`. E.g. it could implement a filter:
```c++
for (int i : v)
{
    if (i % 2 == 0)
        result.push_back(i);
}
```

Here a more expressive version would be:
```c++
copy_if(begin(v), end(v), back_inserter(result), [](int i)
{
    return i % 2 == 0;
});
```
`transform` and `copy_if` show the [map](http://en.wikipedia.org/wiki/Map_%28higher-order_function%29) [filter](http://en.wikipedia.org/wiki/Filter_%28higher-order_function%29) difference more clearly than the two for loops with the same header and just a differing body.

## Performance
"But I have to use the hand written for loop for better performance!" - Nope, you do not have to.
Even if the `std::transform` version looks like much abstraction induced function call overhead, especially with the lambda function, there is none. It is all optimized away by the compiler.

For 100 million values the different implementations ([source code](https://gist.github.com/Dobiasd/839acc2bc7a1f48a5063)) took the following cpu times on my machine:
```
goto            - elapsed time: 0.465151s
while           - elapsed time: 0.464604s
for             - elapsed time: 0.465315s
range based for - elapsed time: 0.463258s
std::transform  - elapsed time: 0.464696s
```

## Conclusion
For better maintainability of your C++ software make use of the cool stuff in the [`<algorithm>` header](http://en.cppreference.com/w/cpp/algorithm). Once you get used to it you will enjoy every for loop you do *not* have to read. ;-)

With [effective stl](http://www.amazon.com/dp/0201749629) Scott Meyers has written a very nice book covering this and more in depths.
Herb Sutter's [talk about lambdas](https://www.youtube.com/watch?v=rcgRY7sOA58) can also help to get more into this topic.