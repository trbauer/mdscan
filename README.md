# mdscan
A simpler markdown scanner (checks various grammar mistakes I make)

Blog plans:
  http://pandoc.org/getting-started.html
  find or write markdown processor (depend on pandoc Haskell package!)
    fundamental problems with pandoc (a) hard to install and (b) no lines in AST

  looks like I'll have to write my own
   use http://spec.commonmark.org/ or
       https://help.github.com/articles/basic-writing-and-formatting-syntax/
       https://help.github.com/articles/creating-and-highlighting-code-blocks/
   mdscan (scans markdown)

     can test via
     http://spec.commonmark.org/dingus/?text=-%20foo%0A-%0A-%20bar

   PP
     - breaks things into lines
     - tabs to 4 spcs
     - emptpy line is called blank line
   Blocks
     - contain Inlines
     - take syntactic precendence
     - can be leaf or container

   data Block =
      BlHr -- 4.1 horizontal rule
      -- e.g. ----  => <hr />
      -- 0-3 spaces of indentation followed by 3 or more -, _, or *
      -- each followed by any number of spaces
      -- 4 spaces of indent implies code
      -- my strict def could just assume 0 spaces
    | BlAtx String -- 4.2
      -- e.g.  # foo => <h1>foo</h1>
      --       ## foo => <h2>...

      -- 1 to 6 #'s <h1> to <h6> followed by space followed by non-space
      -- leading and trailing blanks are stripped
      -- optional closing #'s of any count (and spaces)
      -- indentation is permitted, but I will disallow
      -- headers can contain the empty string as the title

    | BlSext String -- setext header
      -- line with at least one non-ws char followed by underlinging
      -- Foo *bar*
      -- =====
      -- "=" => level 1 (h1), "-" implies level 2
      -- canot interrupt a pg
    | BlIndCode String -- 4.4 indented code block
      -- end paragraph
      --
      --     code
      -- * text internally is taken literally
      -- * may not interrupt paragraphs
    | BlFencedCode String -- 4.5 fenced code block
      -- ```            => <pre><code>
      --   foo>`            foo&gt;`
      -- ```               </code></pre>
      -- * also ~~~ is allowable
      -- * opening and closing fence must be same length (actually >, but I'm strict)
      -- * unclosed blocks consume to EOF (I'm rejecting this)
      -- * may interrupt paragraphs
      -- * info string informs the code class
      --   ~~~haskell  ==> <pre><code class="language-haskell">...
    | BlHtml -- 4.6 html code block
    | BlLinkRef -- 4.7
    | BlPara -- 4.8
      -- * sequence of non-blank lines that cannot be interpreted as other blocks
      -- * can be separated by 1 or more lines (same effect)
    -- 4.9 blank lines

    -- 5.X CONTAINER BLOCKS

    | BlQuote [Indent] -- 5.1 block quote
    -- * prefixed with >
    -- * ==> <blockquote>
    -- e.g. normal
    --      > # quoted title
    --      > bar
    -- * fairly complicated rules
    | BlListItems (String,Block) -- 5.2 bullet/ordered list (fst==bullet)
    -- * opned by bullet list marker
    --    -, + , or *
    -- * ordered list marker
    --     [0-9]+[.)]
    | BlList [Block]

-- 6.0
data Inline

    -- 6.1 escapes
    -- 6.2 entities (must convert the usual html " & < > &quot; &amp; &lt; &gt;
   InCodeSpan -- 6.3 code span
   -- the code `foo` works. => <code>
   -- * if no closing `, then EOL suffices (otherwise use \` for real \`89)
 | InEmph -- 6.4 emphasis and strong em
   --   *foo* => <em>
   --   **foo** => <strong>
   -- * emphasis can continue over line breaks!
   -- * e.g. * foo
   --        bar*
   -- * emphasis can contain other constructs (e.g. links)
