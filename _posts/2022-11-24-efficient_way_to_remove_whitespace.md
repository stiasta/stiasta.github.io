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
