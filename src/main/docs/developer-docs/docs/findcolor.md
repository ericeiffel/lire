# Searching for Color with Lire

Since Lire already supports Color histograms (with the MPEG-7 ScalableColor descriptor), 
functions for searching for colors have been added in Lire 0.5.1:

  * With ``DocumentBuilderFactory.getColorOnlyDocumentBuilder()`` you will get an appropriate DocumentBuilder for indexing the images with Scalable Color only. You can also use the ``DocumentBuilderFactory.getExtensiveDocumentBuilder()``, then all information for texture and color distribution search is included.
  * With ``DocumentFactory.createColorOnlyDocument(java.awt.Color)`` a simple document for searching based on one color can be created in a fast and efficient way.
  * Note that the whole images are indexed. A far better way would be to distinguish between background and foreground. However this topic is currently not addressed in Lire.

Contributions for background foreground separation are welcome :-)

Another option for color only searching is using the [[lire:AutoColorCorrelation]] descriptor (included currently only in CVS). This descriptor uses the color information of images as well as the correlation of colors in the image. See [[lire:AutoColorCorrelation]] for more details.

## Sample Code: Indexing 
This is a code sample showing how the indexing should be done:

    public class CreateIndexTest extends TestCase {
        private String[] testFiles = new String[]{"img01.JPG", "img02.JPG", "img03.JPG", "img04.JPG", "img05.JPG",
                "img06.JPG", "img07.JPG", "img08.JPG", "img08a.JPG"};
        private String testFilesPath = "./src/test/resources/images/";
        private String indexPath = "test-index";
    
        public void testCreateIndex() throws IOException {
            DocumentBuilder builder = DocumentBuilderFactory.getColorOnlyDocumentBuilder();
            // The alternative way is using the extensive DocumentBuilder:
            // DocumentBuilder builder = DocumentBuilderFactory.getExtensiveDocumentBuilder();
            IndexWriter iw = new IndexWriter(FSDirectory.open(new File(indexPath + "-small")), new SimpleAnalyzer(), true, IndexWriter.MaxFieldLength.UNLIMITED);
            for (String identifier : testFiles) {
                Document doc = builder.createDocument(new FileInputStream(testFilesPath + identifier), identifier);
                iw.addDocument(doc);
            }
            iw.optimize();
            iw.close();
        }
    }

## Sample Code: Searching 
This is a code sample showing how the search can be done:

    public class TestImageSearcher extends TestCase {
        private String indexPath = "test-index-small";

        /**
         * Tests the search for color ...
         * Please note that the index has to be built with a DocumentBuilder
         * obtained by DocumentBuilderFactory.getExtensiveDocumentBuilder() or
         * DocumentBuilderFactory.getColorOnlyDocumentBuilder().
         * @throws Exception in case of exceptions *gg*
         */
        public void testColorSearch() throws Exception {
            // open index with Lucene v3.0+
            IndexReader reader = IndexReader.open(FSDirectory.open(new File(indexPath)));
            int numDocs = reader.numDocs();
            System.out.println("numDocs = " + numDocs);
            // PLEASE NOTE: The index has to be built with a DocumentBuilder
            // obtained by DocumentBuilderFactory.getExtensiveDocumentBuilder() or
            // DocumentBuilderFactory.getColorOnlyDocumentBuilder(). Otherwise no color
            // histogram will be included.
            ImageSearcher searcher = ImageSearcherFactory.createColorOnlySearcher(5);
            Document document = DocumentFactory.createColorOnlyDocument(Color.red);
            ImageSearchHits hits = searcher.search(document, reader);
            for (int i = 0; i < hits.length(); i++) {
                System.out.println(hits.score(i) + ": " +
                    hits.doc(i).getField(DocumentBuilder.FIELD_NAME_IDENTIFIER).stringValue());
            }
        }
    }