---
layout: post
title: Efficient way to remove ALL whitespace from String?
description: Fastest and most efficient way to remove ALL whitespace from String?
categories: [programming, c#, dotnet, optimization]
---

I have had a [StackOverflow answer](https://stackoverflow.com/questions/6219454/efficient-way-to-remove-all-whitespace-from-string/37347881#37347881) to the question "Efficient way to remove ALL whitespace from String?" for a while now. After [.NET 7 was released](https://github.com/dotnet/core/blob/main/release-notes/7.0/7.0.0/7.0.0.md?WT.mc_id=dotnet-35129-website) with many performance optimizations I figured I wanted to go over this answer once again testing all of the "remove whitespace" methods up against multiple frameworks. To make the test a little more robust I used [BenchmarkDotnet](https://benchmarkdotnet.org/) library. 

The runtimes I added to this test was 
- .NETcore 3.1
- .NET 6
- .NET 7

# Benchmark code
```C#
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;

using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Jobs;

namespace RemoveWhitespaceBenchmark
{
    [SimpleJob(RuntimeMoniker.NetCoreApp31)]
    [SimpleJob(RuntimeMoniker.Net60)]
    [SimpleJob(RuntimeMoniker.Net70)]
    public class RemoveWhitespace
    {
        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveStringReader(string text, string name)
        {
            StringBuilder s = new StringBuilder(text.Length);
            using (StringReader reader = new StringReader(text))
            {
                int i = 0;
                char c;
                for (; i < text.Length; i++)
                {
                    c = (char)reader.Read();
                    if (!char.IsWhiteSpace(c))
                    {
                        s.Append(c);
                    }
                }
            }

            return s.ToString();
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveLinqNativeCharIsWhitespace(string text, string name)
        {
            return new string(text.ToCharArray()
                .Where(c => !char.IsWhiteSpace(c))
                .ToArray());
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveLinq(string text, string name)
        {
            return new string(text.ToCharArray()
                .Where(c => !Char.IsWhiteSpace(c))
                .ToArray());
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveRegex(string text, string name)
        {
            return Regex.Replace(text, @"\s+", "");
        }

        private static readonly Regex compiled = new Regex(@"\s+", RegexOptions.Compiled);

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveRegexCompiled(string text, string name)
        {
            return compiled.Replace(text, "");
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveForLoop(string text, string name)
        {
            string input = text;
            for (int i = input.Length - 1; i >= 0; i--)
            {
                if (char.IsWhiteSpace(input[i]))
                {
                    input = input.Remove(i, 1);
                }
            }
            return input;
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string StringSplitThenJoin(string text, string name)
        {
            return string.Join("", text.Split(default(string[]), StringSplitOptions.RemoveEmptyEntries));
        }

        [Benchmark]
        [ArgumentsSource(nameof(Data))]
        public string RemoveInPlaceCharArray(string text, string name)
        {
            int len = text.Length;
            char[] src = text.ToCharArray();
            int dstIdx = 0;
            for (int i = 0; i < len; i++)
            {
                char ch = src[i];
                switch (ch)
                {
                    case '\u0020':
                    case '\u00A0':
                    case '\u1680':
                    case '\u2000':
                    case '\u2001':
                    case '\u2002':
                    case '\u2003':
                    case '\u2004':
                    case '\u2005':
                    case '\u2006':
                    case '\u2007':
                    case '\u2008':
                    case '\u2009':
                    case '\u200A':
                    case '\u202F':
                    case '\u205F':
                    case '\u3000':
                    case '\u2028':
                    case '\u2029':
                    case '\u0009':
                    case '\u000A':
                    case '\u000B':
                    case '\u000C':
                    case '\u000D':
                    case '\u0085':
                        continue;
                    default:
                        src[dstIdx++] = ch;
                        break;
                }
            }
            return new string(src, 0, dstIdx);
        }

        // Short input
        private const string SHORT_TEXT = "123 123 \t 1adc \n 222";

        private const string EXPECTED_SHORT_TEXT = "1231231adc222";

        // Long input
        private const string LONG_TEXT = "123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222123 123 \t 1adc \n 222";

        private const string EXPECTED_LONG_TEXT = "1231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc2221231231adc222";

        public IEnumerable<object[]> Data()
        {
            yield return new object[] { SHORT_TEXT, "SHORT" };
            yield return new object[] { LONG_TEXT, "LONG" };
        }
    }
}
```

*Important:* The `.csproj` file has to look something like this: 
```XML
<PropertyGroup>
  <TargetFrameworks>netcoreapp3.1;net6.0;net7.0</TargetFrameworks>
  <ImplicitUsings>disable</ImplicitUsings>
  <GenerateProgramFile>false</GenerateProgramFile>
  [...]
</PropertyGroup>
```
Since it is older versions of C#, implicit usings is not a feature.

# Running the benchmark test
To run the benchmark you only have to add this to your `Program.cs` file
```C#
public static void Main(string[] args)
{
    BenchmarkRunner.Run<RemoveWhitespace>();
}
```

# Results
## Short Text
|                           Method |           Job |       Runtime |                 text |  name |         Mean |      Error |     StdDev |       Median |
|----------------------------------|---------------|---------------|----------------------|-------|-------------:|-----------:|-----------:|-------------:|
|               RemoveStringReader |      .NET 6.0 |      .NET 6.0 | 123 123     1adc 222 | SHORT |    122.86 ns |   2.435 ns |   2.158 ns |    122.70 ns |
| RemoveLinqNativeCharIsWhitespace |      .NET 6.0 |      .NET 6.0 | 123 123     1adc 222 | SHORT |    196.58 ns |   3.864 ns |   4.134 ns |    195.19 ns |
|                       RemoveLinq |      .NET 6.0 |      .NET 6.0 | 123 123     1adc 222 | SHORT |    208.75 ns |   6.480 ns |  17.740 ns |    204.15 ns |
|                      RemoveRegex |      .NET 6.0 |      .NET 6.0 | 123 123     1adc 222 | SHORT |    505.46 ns |   9.457 ns |   8.846 ns |    501.08 ns |
|              RemoveRegexCompiled |      .NET 6.0 |      .NET 6.0 | 123 123     1adc 222 | SHORT |    275.37 ns |   4.940 ns |   5.490 ns |    274.29 ns |
|                    RemoveForLoop |      .NET 6.0 |      .NET 6.0 | 123 123     1adc 222 | SHORT |    145.00 ns |   6.990 ns |  19.829 ns |    138.66 ns |
|              StringSplitThenJoin |      .NET 6.0 |      .NET 6.0 | 123 123     1adc 222 | SHORT |    195.59 ns |   3.962 ns |   6.050 ns |    195.68 ns |
|           RemoveInPlaceCharArray |      .NET 6.0 |      .NET 6.0 | 123 123     1adc 222 | SHORT |     56.17 ns |   1.601 ns |   4.516 ns |     55.45 ns |
|               RemoveStringReader |      .NET 7.0 |      .NET 7.0 | 123 123     1adc 222 | SHORT |     96.39 ns |   2.538 ns |   7.282 ns |     95.44 ns |
| RemoveLinqNativeCharIsWhitespace |      .NET 7.0 |      .NET 7.0 | 123 123     1adc 222 | SHORT |    214.77 ns |   5.211 ns |  15.035 ns |    209.56 ns |
|                       RemoveLinq |      .NET 7.0 |      .NET 7.0 | 123 123     1adc 222 | SHORT |    205.01 ns |   4.172 ns |   9.502 ns |    203.98 ns |
|                      RemoveRegex |      .NET 7.0 |      .NET 7.0 | 123 123     1adc 222 | SHORT |    550.95 ns |   9.963 ns |   8.832 ns |    548.10 ns |
|              RemoveRegexCompiled |      .NET 7.0 |      .NET 7.0 | 123 123     1adc 222 | SHORT |    314.28 ns |   6.207 ns |   8.902 ns |    313.59 ns |
|                    RemoveForLoop |      .NET 7.0 |      .NET 7.0 | 123 123     1adc 222 | SHORT |    158.74 ns |   5.834 ns |  17.110 ns |    152.90 ns |
|              StringSplitThenJoin |      .NET 7.0 |      .NET 7.0 | 123 123     1adc 222 | SHORT |    202.84 ns |   4.001 ns |   3.742 ns |    201.46 ns |
|           RemoveInPlaceCharArray |      .NET 7.0 |      .NET 7.0 | 123 123     1adc 222 | SHORT |     59.29 ns |   1.221 ns |   1.751 ns |     58.68 ns |
|               RemoveStringReader | .NET Core 3.1 | .NET Core 3.1 | 123 123     1adc 222 | SHORT |    156.72 ns |   3.024 ns |   3.361 ns |    156.18 ns |
| RemoveLinqNativeCharIsWhitespace | .NET Core 3.1 | .NET Core 3.1 | 123 123     1adc 222 | SHORT |    246.68 ns |   4.885 ns |   5.999 ns |    244.60 ns |
|                       RemoveLinq | .NET Core 3.1 | .NET Core 3.1 | 123 123     1adc 222 | SHORT |    246.52 ns |   2.866 ns |   2.541 ns |    247.07 ns |
|                      RemoveRegex | .NET Core 3.1 | .NET Core 3.1 | 123 123     1adc 222 | SHORT |  1,199.94 ns |  22.786 ns |  22.379 ns |  1,189.29 ns |
|              RemoveRegexCompiled | .NET Core 3.1 | .NET Core 3.1 | 123 123     1adc 222 | SHORT |  1,007.07 ns |  19.556 ns |  25.429 ns |  1,004.26 ns |
|                    RemoveForLoop | .NET Core 3.1 | .NET Core 3.1 | 123 123     1adc 222 | SHORT |    155.61 ns |   3.025 ns |   3.483 ns |    154.43 ns |
|              StringSplitThenJoin | .NET Core 3.1 | .NET Core 3.1 | 123 123     1adc 222 | SHORT |    238.61 ns |   3.902 ns |   3.258 ns |    237.57 ns |
|           RemoveInPlaceCharArray | .NET Core 3.1 | .NET Core 3.1 | 123 123     1adc 222 | SHORT |     53.05 ns |   1.105 ns |   1.085 ns |     52.72 ns |

## Long Text
|                           Method |           Job |       Runtime |                 text |  name |         Mean |      Error |     StdDev |       Median |
|--------------------------------- |-------------- |-------------- |--------------------- |------ |-------------:|-----------:|-----------:|-------------:|
|               RemoveStringReader |      .NET 6.0 |      .NET 6.0 |  123 (...) 222 [440] |  LONG |  2,207.21 ns |  42.806 ns |  57.144 ns |  2,186.48 ns |
| RemoveLinqNativeCharIsWhitespace |      .NET 6.0 |      .NET 6.0 |  123 (...) 222 [440] |  LONG |  2,090.81 ns |  19.523 ns |  15.242 ns |  2,085.87 ns |
|                       RemoveLinq |      .NET 6.0 |      .NET 6.0 |  123 (...) 222 [440] |  LONG |  2,131.09 ns |  38.736 ns |  54.302 ns |  2,113.29 ns |
|                      RemoveRegex |      .NET 6.0 |      .NET 6.0 |  123 (...) 222 [440] |  LONG |  8,029.84 ns | 148.297 ns | 123.835 ns |  8,014.60 ns |
|              RemoveRegexCompiled |      .NET 6.0 |      .NET 6.0 |  123 (...) 222 [440] |  LONG |  3,673.86 ns |  58.893 ns |  49.179 ns |  3,665.01 ns |
|                    RemoveForLoop |      .NET 6.0 |      .NET 6.0 |  123 (...) 222 [440] |  LONG |  9,574.58 ns | 170.786 ns | 250.336 ns |  9,560.20 ns |
|              StringSplitThenJoin |      .NET 6.0 |      .NET 6.0 |  123 (...) 222 [440] |  LONG |  2,872.59 ns |  56.242 ns |  75.082 ns |  2,846.50 ns |
|           RemoveInPlaceCharArray |      .NET 6.0 |      .NET 6.0 |  123 (...) 222 [440] |  LONG |    795.89 ns |  15.824 ns |  26.000 ns |    789.73 ns |
|               RemoveStringReader |      .NET 7.0 |      .NET 7.0 |  123 (...) 222 [440] |  LONG |  1,239.48 ns |  22.250 ns |  23.807 ns |  1,234.84 ns |
| RemoveLinqNativeCharIsWhitespace |      .NET 7.0 |      .NET 7.0 |  123 (...) 222 [440] |  LONG |  2,162.54 ns |  40.383 ns |  49.594 ns |  2,144.46 ns |
|                       RemoveLinq |      .NET 7.0 |      .NET 7.0 |  123 (...) 222 [440] |  LONG |  2,127.38 ns |  38.694 ns |  48.936 ns |  2,110.64 ns |
|                      RemoveRegex |      .NET 7.0 |      .NET 7.0 |  123 (...) 222 [440] |  LONG |  8,064.66 ns | 153.110 ns | 150.375 ns |  8,044.91 ns |
|              RemoveRegexCompiled |      .NET 7.0 |      .NET 7.0 |  123 (...) 222 [440] |  LONG |  3,858.84 ns |  74.988 ns |  80.236 ns |  3,834.91 ns |
|                    RemoveForLoop |      .NET 7.0 |      .NET 7.0 |  123 (...) 222 [440] |  LONG |  9,534.81 ns | 187.211 ns | 229.912 ns |  9,450.11 ns |
|              StringSplitThenJoin |      .NET 7.0 |      .NET 7.0 |  123 (...) 222 [440] |  LONG |  2,860.38 ns |  48.188 ns |  42.717 ns |  2,849.34 ns |
|           RemoveInPlaceCharArray |      .NET 7.0 |      .NET 7.0 |  123 (...) 222 [440] |  LONG |    860.86 ns |  17.298 ns |  26.416 ns |    851.84 ns |
|               RemoveStringReader | .NET Core 3.1 | .NET Core 3.1 |  123 (...) 222 [440] |  LONG |  2,925.74 ns |  54.859 ns | 106.999 ns |  2,893.67 ns |
| RemoveLinqNativeCharIsWhitespace | .NET Core 3.1 | .NET Core 3.1 |  123 (...) 222 [440] |  LONG |  2,251.41 ns |  43.370 ns |  46.405 ns |  2,234.34 ns |
|                       RemoveLinq | .NET Core 3.1 | .NET Core 3.1 |  123 (...) 222 [440] |  LONG |  2,467.73 ns |  31.776 ns |  24.809 ns |  2,475.30 ns |
|                      RemoveRegex | .NET Core 3.1 | .NET Core 3.1 |  123 (...) 222 [440] |  LONG | 23,199.52 ns | 297.498 ns | 232.267 ns | 23,290.44 ns |
|              RemoveRegexCompiled | .NET Core 3.1 | .NET Core 3.1 |  123 (...) 222 [440] |  LONG | 20,208.25 ns | 302.477 ns | 268.138 ns | 20,191.36 ns |
|                    RemoveForLoop | .NET Core 3.1 | .NET Core 3.1 |  123 (...) 222 [440] |  LONG | 10,212.44 ns | 200.690 ns | 187.725 ns | 10,218.33 ns |
|              StringSplitThenJoin | .NET Core 3.1 | .NET Core 3.1 |  123 (...) 222 [440] |  LONG |  3,833.31 ns |  59.144 ns |  63.283 ns |  3,844.23 ns |
|           RemoveInPlaceCharArray | .NET Core 3.1 | .NET Core 3.1 |  123 (...) 222 [440] |  LONG |    865.46 ns |  29.865 ns |  85.206 ns |    842.39 ns |
