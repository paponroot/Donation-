import java.awt.*;
import java.io.*;
import java.util.*;
import javax.swing.*;

abstract class User {
    protected String name, email, phone;

    public User(String name, String email, String phone) {
        this.name = name;
        this.email = email;
        this.phone = phone;
    }

    public abstract String getRole();
}

class Admin extends User {
    public Admin() {
        super("Admin", "", "");
    }

    public String getRole() {
        return "Admin";
    }
}

class Member extends User {
    public Member(String name, String email, String phone) {
        super(name, email, phone);
    }

    public String getRole() {
        return "Member";
    }
}

class ClothingItem {
    private String id, category, status;
    private String donorName, donorEmail, donorPhone;

    public ClothingItem(String id, String category,
                        String name, String email, String phone) {
        this.id = id;
        this.category = category;
        this.status = "Available";
        this.donorName = name;
        this.donorEmail = email;
        this.donorPhone = phone;
    }

    public String toFileString() {
        return id + " ." + category + " ." + status + " ." +
               donorName + " ." + donorEmail + " ." + donorPhone;
    }
}

class FileRepository {
    private final String file = "items.txt";

    public void addItem(ClothingItem item) {
        try (PrintWriter w = new PrintWriter(new FileWriter(file, true))) {
            w.println(item.toFileString());
        } catch (Exception e) {
            JOptionPane.showMessageDialog(null, "Error saving item");
        }
    }

    public java.util.List<String> getItems() {
        java.util.List<String> list = new ArrayList<>();
        try (BufferedReader r = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = r.readLine()) != null) list.add(line);
        } catch (Exception e) {
            JOptionPane.showMessageDialog(null, "Error reading file");
        }
        return list;
    }

    public void deleteItem(String id) {
        java.util.List<String> list = getItems();
        try (PrintWriter w = new PrintWriter(file)) {
            for (String s : list) {
                if (!s.startsWith(id + " .")) w.println(s);
            }
        } catch (Exception e) {
            JOptionPane.showMessageDialog(null, "Delete failed");
        }
    }
}

class MainFrame extends JFrame {
    private FileRepository repo = new FileRepository();
    private JTextArea area;
    private User currentUser;

    public MainFrame() {
        login();

        setTitle("Donation System - " + currentUser.getRole());
        setSize(650, 500);
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        JPanel panel = new JPanel();

        JButton donateBtn = new JButton("Donate Item");
        panel.add(donateBtn);

        JButton viewBtn = new JButton("View Items");
        JButton deleteBtn = new JButton("Delete Item");

        if (currentUser.getRole().equals("Admin")) {
            panel.add(viewBtn);
            panel.add(deleteBtn);
        }

        area = new JTextArea();
        JScrollPane scroll = new JScrollPane(area);

        ImageIcon icon = new ImageIcon("in-some-3110417_640.jpg");
        Image img = icon.getImage().getScaledInstance(650, 150, Image.SCALE_SMOOTH);
        JLabel imageLabel = new JLabel(new ImageIcon(img));

        JPanel topPanel = new JPanel(new BorderLayout());
        topPanel.add(imageLabel, BorderLayout.NORTH);
        topPanel.add(panel, BorderLayout.SOUTH);

        setLayout(new BorderLayout());
        add(topPanel, BorderLayout.NORTH);
        add(scroll, BorderLayout.CENTER);

        donateBtn.addActionListener(e -> donateItem());

        if (currentUser.getRole().equals("Admin")) {
            viewBtn.addActionListener(e -> loadItems());
            deleteBtn.addActionListener(e -> deleteItem());
        }

        setVisible(true);
    }

    private void login() {
        String[] roles = {"Admin", "Member"};

        String choice = (String) JOptionPane.showInputDialog(
                null, "Login as", "Login",
                JOptionPane.QUESTION_MESSAGE, null,
                roles, roles[0]);

        if (choice == null) System.exit(0);

        if (choice.equals("Admin")) {
            String pass = JOptionPane.showInputDialog("Enter admin password");
            if (!"admin123".equals(pass)) {
                JOptionPane.showMessageDialog(null, "Wrong password");
                System.exit(0);
            }
            currentUser = new Admin();
        } else {
            String name = JOptionPane.showInputDialog("Enter name");
            String email = JOptionPane.showInputDialog("Enter email");
            String phone = JOptionPane.showInputDialog("Enter phone");

            currentUser = new Member(name, email, phone);
        }
    }

    private void donateItem() {
        String[] options = {"T-Shirt", "Pant", "Shirt", "Food", "Other"};

        String category = (String) JOptionPane.showInputDialog(
                this,
                "Select item",
                "Donation",
                JOptionPane.QUESTION_MESSAGE,
                null,
                options,
                options[0]
        );

        if (category == null) return;

        String name = JOptionPane.showInputDialog("Enter your name : ");
        String email = JOptionPane.showInputDialog("Enter your email : ");
        String phone = JOptionPane.showInputDialog("Enter your phone : ");

        if (name.isEmpty() || email.isEmpty() || phone.isEmpty()) {
            JOptionPane.showMessageDialog(this, "All fields required");
            return;
        }

        String id = "I" + System.currentTimeMillis();

        ClothingItem item = new ClothingItem(id, category, name, email, phone);
        repo.addItem(item);

        JOptionPane.showMessageDialog(this, "Donation successful");
    }

    private void loadItems() {
        area.setText("");
        for (String s : repo.getItems()) {
            area.append(s + "\n");
        }
    }

    private void deleteItem() {
        String id = JOptionPane.showInputDialog("Enter item id :");
        if (id == null) return;

        repo.deleteItem(id);
        loadItems();
    }
}

public class Project {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new MainFrame());
    }
}
