# WpfSearchAndHighlightText
Search and highlight text in wpf (Rich Text Box)

This is a simple Wpf app, where it has a text box to enter the text we want to search.
The app contain a rich text box, where you can post your text.
Enter the text to search and click 'Search' button.
This will find all the matches and highlights them in yellow.

Also, in status bar at bottom, it will show how many matched found or no matches found.

Method used to search and highlight: (Upon Search button click)

        private void bnt_Search_Click(object sender, RoutedEventArgs e)
        {
            TextRange textRange = new TextRange(rich.Document.ContentStart, rich.Document.ContentEnd);

            //clear up highlighted text before starting a new search
            textRange.ClearAllProperties();
            lbl_Status.Content = "";

           //get the richtextbox text
           string textBoxText = textRange.Text;

            //get search text
            string searchText = searchTextBox.Text;

            if (string.IsNullOrWhiteSpace(textBoxText) || string.IsNullOrWhiteSpace(searchText))
            {
                lbl_Status.Content = "Please provide search text or source text to search from";
            }

            else
            {

                //using regex to get the search count
                //this will include search word even it is part of another word
                //say we are searching "hi" in "hi, how are you Mahi?" --> match count will be 2 (hi in 'Mahi' also)

                Regex regex = new Regex(searchText);
                int count_MatchFound = Regex.Matches(textBoxText, regex.ToString()).Count;

                for (TextPointer startPointer = rich.Document.ContentStart;
                            startPointer.CompareTo(rich.Document.ContentEnd) <= 0;
                                startPointer = startPointer.GetNextContextPosition(LogicalDirection.Forward))
                {
                    //check if end of text
                    if (startPointer.CompareTo(rich.Document.ContentEnd) == 0)
                    {
                        break;
                    }

                    //get the adjacent string
                    string parsedString = startPointer.GetTextInRun(LogicalDirection.Forward);

                    //check if the search string present here
                    int indexOfParseString = parsedString.IndexOf(searchText);

                    if (indexOfParseString >= 0) //present
                    {
                        //setting up the pointer here at this matched index
                        startPointer = startPointer.GetPositionAtOffset(indexOfParseString);

                        if (startPointer != null)
                        {
                            //next pointer will be the length of the search string
                            TextPointer nextPointer = startPointer.GetPositionAtOffset(searchText.Length);

                            //create the text range
                            TextRange searchedTextRange = new TextRange(startPointer, nextPointer);

                            //color up 
                            searchedTextRange.ApplyPropertyValue(TextElement.BackgroundProperty, new  SolidColorBrush(Colors.Yellow));

                            //add other setting property

                        }
                    }
                }

                //update the label text with count
                if (count_MatchFound > 0)
                {
                    lbl_Status.Content = "Total Match Found : " + count_MatchFound;
                }
                else
                {
                    lbl_Status.Content = "No Match Found !";
                }
            }
        }

