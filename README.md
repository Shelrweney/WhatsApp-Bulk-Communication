# WhatsApp-Bulk-Communication
#WhatsApp-Bulk-Communication is Application using in sending at the same time bulk encrypted files secured by password like PDF files, And each employee can access by his own password in his private WhatsApp account
# WhatsApp-Bulk-Communication is written by JAVA 
package jumpstart;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Arrays;
import java.util.Date;
import java.io.IOException;

import com.itextpdf.text.Document;
import com.itextpdf.text.pdf.PdfDocument;
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.Rectangle;

import com.itextpdf.text.Paragraph;
import com.itextpdf.text.pdf.PdfWriter;
import com.itextpdf.text.pdf.PdfReader;
import com.itextpdf.text.pdf.PdfStamper;
import com.itextpdf.text.exceptions.InvalidPdfException;


import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.Reader;
import org.apache.commons.codec.binary.Base64;

import com.google.gson.Gson;

public class WaPdfSender {

    /**
     * Inner class that captures the information needed to construct the JSON object
     * for sending a Document message.
     */
    class DocMessage {
        String number = null;
        String document = null;
        String filename = null;
    }
    
    // TODO: Replace the following with your gateway instance ID, Forever Green
    // Client ID and Secret below.
    private static final String INSTANCE_ID = "INSTANCE_ID";
    private static final String CLIENT_ID = "ClientID";
    private static final String CLIENT_SECRET = "Client secret";

    private static final String GATEWAY_URL = "http://api.whatsmate.net/v3/whatsapp/single/document/message/" + INSTANCE_ID;

    /**
     * Entry Point
     */
    public static void main(String[] args) throws Exception {
       
    	
    	File dir = new File("D:\\private\\PDfR") ;
    	//String user = "Hello";
	      // String owner = "World";
	       String[] SRC =dir.list();
	       String [] password = {"world1","world2","world3","world4"};
	       String[] empcode= {"0","2","6","1"};
	       
	       
	       for (int i=0;i< empcode.length;i++) {
    		
	    	   if((Arrays.asList(SRC).contains("studentstatuscertificates"+empcode[i]+".pdf"))) {
System.out.println("studentstatuscertificates"+empcode[i]+".pdf");	
    	
 	 //tring DEST ="D:\\private\\NewPDF\\"+(SRC[i]);
 	 //le file = new File(DEST);
       //ile.getParentFile().mkdirs();
        
    	PdfReader reader = new PdfReader("D:\\private\\PDfR\\"+"studentstatuscertificates"+empcode[i]+".pdf");
        PdfStamper stamper = new PdfStamper(reader, new FileOutputStream("D:\\private\\NewPDF\\"+"studentstatuscertificates"+empcode[i]+".pdf"));
        
		stamper.setEncryption(password[i].getBytes(), password[i].getBytes(),
            PdfWriter.ALLOW_PRINTING, PdfWriter.ENCRYPTION_AES_128 | PdfWriter.DO_NOT_ENCRYPT_METADATA);
       
        stamper.close();
	    reader.close();

    	
    	
    	///////////////////////////////////////////WatsApp code/////////////////////////////////////////////////
    	
    	
    	// TODO: Specify the recipients of your image 
        String [] recipient = {"PhoneNumber1","","PhoneNumber2","+PhoneNumber3"};
        // TODO: Specify the content of your image
        Path docPath = Paths.get("D:\\private\\NewPDF\\"+"studentstatuscertificates"+empcode[i]+".pdf");
        byte[] docBytes = Files.readAllBytes(docPath);
       
        String filename = ("studentstatuscertificates"+empcode[i]+".pdf");
        System.out.println(filename);	
        
        WaPdfSender docSender = new WaPdfSender();
        docSender.sendDocMessage(recipient[i], docBytes, filename);

    	}}}
    /**
     * Sends out a WhatsApp message (a document) to a person
     */
    public void sendDocMessage(String recipient, byte[] docBytes, String filename)
            throws Exception {
        byte[] encodedBytes = Base64.encodeBase64(docBytes);
        String base64Doc = new String(encodedBytes);
        //System.out.println(base64Doc);
        DocMessage docMsgObj = new DocMessage();
        docMsgObj.number = recipient;
        docMsgObj.document = base64Doc;
        docMsgObj.filename = filename;

        Gson gson = new Gson();
        String jsonPayload = gson.toJson(docMsgObj);

        URL url = new URL(GATEWAY_URL);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setDoOutput(true);
        conn.setRequestMethod("POST");
        conn.setRequestProperty("X-WM-CLIENT-ID", CLIENT_ID);
        conn.setRequestProperty("X-WM-CLIENT-SECRET", CLIENT_SECRET);
        conn.setRequestProperty("Content-Type", "application/json");

        OutputStream os = conn.getOutputStream();
        os.write(jsonPayload.getBytes());
        os.flush();
        os.close();

        int statusCode = conn.getResponseCode();
        System.out.println("Response from WhatsApp Gateway: \n");
        System.out.println("Status Code: " + statusCode);
        BufferedReader br = new BufferedReader(new InputStreamReader(
                (statusCode == 200) ? conn.getInputStream()
                        : conn.getErrorStream()));
        String output;
        while ((output = br.readLine()) != null) {
            System.out.println(output);
        }
        conn.disconnect();
    }

}

