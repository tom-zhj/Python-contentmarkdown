# 机器学习之nltk download安装测试包

接着上一篇文章 机器学习之nltk download出错：Error connecting to server: [Errno -2] ，下面说一下
nltk测试包的安装及要注意的事项

>>> import nltk

>>> nltk.download()

NLTK Downloader

\---------------------------------------------------------------------------

   d) Download      l) List      c) Config      h) Help      q) Quit

\---------------------------------------------------------------------------

Downloader> d

Download which package (l=list; x=cancel)?

 Identifier>

\---------------------------------------------------------------------------

   d) Download      l) List      c) Config      h) Help      q) Quit

\---------------------------------------------------------------------------

这里要注意：这一步的时候要选择l（list）

Downloader> l

Packages:

 [ ] brown_tei........... Brown Corpus (TEI XML Version)

 [ ] punkt............... Punkt Tokenizer Models

 [ ] maxent_treebank_pos_tagger Treebank Part of Speech Tagger (Maximum
entropy)

 [ ] machado............. Machado de Assis -- Obra Completa

 [ ] movie_reviews....... Sentiment Polarity Dataset Version 2.0

 [ ] names............... Names Corpus, Version 1.3 (1994-03-29)

 [ ] nombank.1.0......... NomBank Corpus 1.0

 [ ] nps_chat............ NPS Chat

 [ ] paradigms........... Paradigm Corpus

 [ ] pe08................ Cross-Framework and Cross-Domain Parser

                          Evaluation Shared Task

 [ ] pil................. The Patient Information Leaflet (PIL) Corpus

 [ ] pl196x.............. Polish language of the XX century sixties

 [ ] ppattach............ Prepositional Phrase Attachment Corpus

 [ ] problem_reports..... Problem Report Corpus

 [ ] propbank............ Proposition Bank Corpus 1.0

 [ ] qc.................. Experimental Data for Question Classification

 [ ] reuters............. The Reuters-21578 benchmark corpus, ApteMod

                          version

 [ ] rte................. PASCAL RTE Challenges 1, 2, and 3

Hit Enter to continue:

查看所有的包，并找到你需要的包，然后不能按照提示收入点击，而是应该这样做：

>>> nltk.download('brown_tei')

注意：该方法可能会出现：<urlopen error [Errno -2] Name or service not
known>的错误，这时可使用下面的方法解决

或者使用：

python -m nltk.downloader spanish_grammars

  

