!SLIDE 
# My Presentation #

!SLIDE encode-symbol
# Huffman encoding #

	@@@ clojure
	(defn encode
	  "takes as arguments a message and a tree
	 and produces the list of bits that gives
	 the encoded message"
	[[first & rest :as message] tree]
	  (when (seq message)
	    (concat (encode-symbol first tree)
	            (encode rest tree))))
