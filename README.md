import java.io.IOException;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Scanner;
import org.json.JSONArray;
import org.json.JSONObject;
public class GraphAPIExample {
    private static final String ACCESS_TOKEN = "YOUR_ACCESS_TOKEN";
    private static final String GRAPH_API_URL = "https://graph.microsoft.com/v1.0/me/messages";
    public static void main(String[] args) throws IOException {
        // Retrieve all messages
        String response = getMessages();
        JSONArray messages = new JSONObject(response).getJSONArray("value");
        // Group messages by conversationId
        Map<String, JSONArray> conversations = new HashMap<>();
        for (int i = 0; i < messages.length(); i++) {
            JSONObject message = messages.getJSONObject(i);
            String conversationId = message.getString("conversationId");
            conversations.computeIfAbsent(conversationId, k -> new JSONArray()).put(message);
        }
        // Print grouped messages
        for (Map.Entry<String, JSONArray> entry : conversations.entrySet()) {
            System.out.println("Conversation ID: " + entry.getKey());
            JSONArray msgs = entry.getValue();
            for (int i = 0; i < msgs.length(); i++) {
                JSONObject msg = msgs.getJSONObject(i);
                System.out.println("  Subject: " + msg.getString("subject"));
                System.out.println("  Received: " + msg.getString("receivedDateTime"));
                System.out.println("  From: " + msg.getJSONObject("sender").getJSONObject("emailAddress").getString("address"));
                System.out.println();
            }
        }
    }
    private static String getMessages() throws IOException {
        URL url = new URL(GRAPH_API_URL);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("GET");
        conn.setRequestProperty("Authorization", "Bearer " + ACCESS_TOKEN);
        Scanner scanner = new Scanner(conn.getInputStream());
        StringBuilder response = new StringBuilder();
        while (scanner.hasNext()) {
            response.append(scanner.nextLine());
        }
        scanner.close();
        return response.toString();
    }
}
 
