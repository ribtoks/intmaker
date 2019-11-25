---
title: .NET String Dictionary vs string switch performance
date: 2015-02-18T22:11:26+00:00
author: "Taras Kushnir"
permalink: /net-string-dictionary-vs-string-switch-performance/
image: goats-competition-dispute.jpg
categories:
  - Programming
keywords:
  - .net
  - c++
  - dictionary
  - efficient
  - programming
  - string
  - switch
---
I had a simple task to map a collection of objects with string property. Map function should replace one string property to another from set of 5-6 strings. Existing solution used Dictionary initialized with those hard-coded values. Once upon a time I tried to compare Dictionary with _int_ keys to _int_ switch and int switch was FAR better. It was chess engine so performance mattered.

Now it's a web request with thousands of rows of reply, serialized to json, so performance matters again. I wrote simple program, which generated 10 million instances of my simple class with several properties and mapped this list with both methods. Before that I ensured that both methods were JIT'ed. You can find <a href="https://github.com/Ribtoks/heap/blob/master/PerformanceTests/StringSwitchTest/StringSwitchTest/Program.cs" target="_blank">source code at Github</a>.

Details below..

<!--more-->

I can show two Map methods for the reference here:

```csharp
private static string MapStringBySwitch(string line)
        {
            switch (line)
            {
                case LargeBoobsType:
                    return "test 1";
                case ExtremeBoobsType:
                    return "another unexpected";
                case UserDefinedBoobs:
                    return "numbers";
                case PrivatePersonCleanBoobs:
                    return "12345567";
                case FixedAcceptSSSBoobs:
                    return ";jhibkjkgyufyj";
                default:
                    return "default value here";
            }
        }
```

Dictionary method is here:

```csharp
private static readonly IDictionary<String, String> BoobsTypeNameMap =
            new Dictionary<String, String>
            {
                {"MultipleType", "MultipleType"},

                {LargeBoobsType, "test 1"},
                {ExtremeBoobsType, "another unexpected"},
                {UserDefinedBoobs, "numbers"},
                {PrivatePersonCleanBoobs, "12345567"},
                {FixedAcceptSSSBoobs, ";jhibkjkgyufyj"},
            };

        public static String MapStringByDict(String type)
        {
            string result;
            if (type == null || 
                !BoobsTypeNameMap.TryGetValue(type, out result))
            {
                result = "default value here";
            }

            return result;
        }
```

Null comparison should be in the original, so I preserved it. Null-check added about 100 milliseconds to Dictionary Map on my machine.

Results:

In Debug version Map with Dictionary took around 750 milliseconds on my machine. In Release version Dictionary Map took around 650 milliseconds. I used method <a href="https://msdn.microsoft.com/en-us/library/bb347013%28v=vs.110%29.aspx" target="_blank">TryGetValue()</a> which is, I guess, the most efficient (Remember to minus 100 milliseconds if you do not need null check).

In Debug version Map with switch took around 450 milliseconds on my machine which is not far better than dictionary and is more complex to actually write (Dictionary is more programmer-friendly than giant switch). In Release version Map with switch took around 250 milliseconds which is 3 times faster than Dictionary Map in Debug and **2.5 times faster than Dictionary Map** in Release.

So, if your goal is to build quick app, use such a low-level optimizations.
