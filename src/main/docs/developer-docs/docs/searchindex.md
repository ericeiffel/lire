# Searching with Lire
Use the ``ImageSearcherFactory`` for creating an ``ImageSearcher``, which will retrieve the images from the index. This can be done by calling ``ImageSearcherFactory.createDefaultSearcher()``. The ``ImageSearcher`` will query for an image, given by an ``InputStream`` or a ``BufferedImage``, or a Lucene ``Document`` describing an image, for instance with the method ``search(BufferedImage, IndexReader)`` or ``search(Document, IndexReader)``. 

Please note that the ``ImageSearcher`` uses a Lucene ``IndexReader`` and does the retrieval with a linear search in the index. The results are returned as ``ImageSearchHits`` object, which aims to simulate a Lucene ``Hits`` object.

Note also that the ``IndexSearcher`` only uses image features, which are available in the specific ``Document`` in the index. If documents only have been indexed with the fast ``DocumentBuilder`` there is no ColorHistogram or EdgeHistogram feature available in the indexed documents, only the ColorLayout feature.

## Sample Code for a Simple Search Implementation

    /**
     * Simple image retrieval with Lire
     * @author Mathias Lux, mathias <at> juggle <dot> at
     */
    public class Searcher {
        public static void main(String[] args) throws IOException {
            // Checking if arg[0] is there and if it is an image.
            BufferedImage img = null;
            boolean passed = false;
            if (args.length > 0) {
                File f = new File(args[0]);
                if (f.exists()) {
                    try {
                        img = ImageIO.read(f);
                        passed = true;
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
            if (!passed) {
                System.out.println("No image given as first argument.");
                System.out.println("Run \"Searcher <query image>\" to search for <query image>.");
                System.exit(1);
            }

            IndexReader ir = DirectoryReader.open(FSDirectory.open(new File("index")));
            ImageSearcher searcher = ImageSearcherFactory.createCEDDImageSearcher(10);

            ImageSearchHits hits = searcher.search(img, ir);
            for (int i = 0; i < hits.length(); i++) {
                String fileName = hits.doc(i).getValues(DocumentBuilder.FIELD_NAME_IDENTIFIER)[0];
                System.out.println(hits.score(i) + ": \t" + fileName);
            }
        }
    }
