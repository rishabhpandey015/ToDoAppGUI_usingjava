import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.io.*;

public class ToDoAppGUI {

    private static final String USERS_FILE = "users.txt";
    private static final String TASKS_FILE = "tasks.txt";
    private static String currentUser = null;

    public static void main(String[] args) {
        // Create the main frame
        JFrame frame = new JFrame("To-Do List Application");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 300);

        // Create panels
        JPanel panel = new JPanel(new GridLayout(4, 1));
        JLabel welcomeLabel = new JLabel("Welcome to the To-Do List Application!", SwingConstants.CENTER);
        JButton loginButton = new JButton("Login");
        JButton registerButton = new JButton("Register");

        // Add components to the panel
        panel.add(welcomeLabel);
        panel.add(loginButton);
        panel.add(registerButton);

        // Add panel to the frame
        frame.getContentPane().add(panel);

        // Action listener for the login button
        loginButton.addActionListener(e -> login(frame));

        // Action listener for the register button
        registerButton.addActionListener(e -> register(frame));

        // Set frame visibility
        frame.setVisible(true);
    }

    private static void login(JFrame frame) {
        // Create login dialog
        JTextField usernameField = new JTextField();
        JPasswordField passwordField = new JPasswordField();
        Object[] fields = {
                "Username:", usernameField,
                "Password:", passwordField
        };

        int option = JOptionPane.showConfirmDialog(frame, fields, "Login", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            String username = usernameField.getText();
            String password = new String(passwordField.getPassword());

            if (authenticate(username, password)) {
                currentUser = username;
                JOptionPane.showMessageDialog(frame, "Login successful!");
                openTaskMenu(frame);  // Open task menu after successful login
            } else {
                JOptionPane.showMessageDialog(frame, "Invalid credentials. Please try again.");
            }
        }
    }

    private static boolean authenticate(String username, String password) {
        try (BufferedReader reader = new BufferedReader(new FileReader(USERS_FILE))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] credentials = line.split(":");
                if (credentials[0].equals(username) && credentials[1].equals(password)) {
                    return true;
                }
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(null, "Error reading users file.");
        }
        return false;
    }

    private static void register(JFrame frame) {
        // Create register dialog
        JTextField usernameField = new JTextField();
        JPasswordField passwordField = new JPasswordField();
        Object[] fields = {
                "New Username:", usernameField,
                "New Password:", passwordField
        };

        int option = JOptionPane.showConfirmDialog(frame, fields, "Register", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            String username = usernameField.getText();
            String password = new String(passwordField.getPassword());

            try (BufferedWriter writer = new BufferedWriter(new FileWriter(USERS_FILE, true))) {
                writer.write(username + ":" + password);
                writer.newLine();
                JOptionPane.showMessageDialog(frame, "Registration successful!");
            } catch (IOException e) {
                JOptionPane.showMessageDialog(frame, "Error writing to users file.");
            }
        }
    }

    private static void openTaskMenu(JFrame frame) {
        // Create task management frame
        JFrame taskFrame = new JFrame("To-Do List Menu");
        taskFrame.setSize(400, 300);
        JPanel taskPanel = new JPanel(new GridLayout(5, 1));

        // Add buttons for task management
        JButton addTaskButton = new JButton("Add Task");
        JButton viewTaskButton = new JButton("View Tasks");
        JButton editTaskButton = new JButton("Edit Task");
        JButton addNotesButton = new JButton("Add Notes to Task");
        JButton logoutButton = new JButton("Logout");

        // Add action listeners
        addTaskButton.addActionListener(e -> addTask(taskFrame));
        viewTaskButton.addActionListener(e -> viewTasks(taskFrame));
        editTaskButton.addActionListener(e -> editTask(taskFrame));
        addNotesButton.addActionListener(e -> addNotes(taskFrame));
        logoutButton.addActionListener(e -> {
            currentUser = null;
            taskFrame.dispose();
            frame.setVisible(true);  // Show the login/register window again
        });

        // Add components to task panel
        taskPanel.add(addTaskButton);
        taskPanel.add(viewTaskButton);
        taskPanel.add(editTaskButton);
        taskPanel.add(addNotesButton);
        taskPanel.add(logoutButton);

        // Add panel to frame
        taskFrame.getContentPane().add(taskPanel);
        taskFrame.setVisible(true);
        frame.setVisible(false);  // Hide the login/register window
    }

    private static void addTask(JFrame frame) {
        JTextField titleField = new JTextField();
        JTextField priorityField = new JTextField();
        Object[] fields = {
                "Task Title:", titleField,
                "Task Priority:", priorityField
        };

        int option = JOptionPane.showConfirmDialog(frame, fields, "Add Task", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            String title = titleField.getText();
            String priority = priorityField.getText();

            try (BufferedWriter writer = new BufferedWriter(new FileWriter(TASKS_FILE, true))) {
                writer.write(currentUser + ":" + title + ":" + priority + ":");
                writer.newLine();
                JOptionPane.showMessageDialog(frame, "Task added successfully!");
            } catch (IOException e) {
                JOptionPane.showMessageDialog(frame, "Error writing to tasks file.");
            }
        }
    }

    private static void viewTasks(JFrame frame) {
        StringBuilder tasks = new StringBuilder();
        try (BufferedReader reader = new BufferedReader(new FileReader(TASKS_FILE))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] task = line.split(":");
                if (task[0].equals(currentUser)) {
                    tasks.append("Title: ").append(task[1])
                         .append(", Priority: ").append(task[2])
                         .append(", Notes: ").append(task.length > 3 ? task[3] : "No notes")
                         .append("\n");
                }
            }
            JOptionPane.showMessageDialog(frame, tasks.length() == 0 ? "No tasks found." : tasks.toString());
        } catch (IOException e) {
            JOptionPane.showMessageDialog(frame, "Error reading tasks file.");
        }
    }

    private static void editTask(JFrame frame) {
        JTextField taskTitleField = new JTextField();
        JTextField newPriorityField = new JTextField();
        Object[] fields = {
                "Task Title to Edit:", taskTitleField,
                "New Priority:", newPriorityField
        };

        int option = JOptionPane.showConfirmDialog(frame, fields, "Edit Task", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            String taskTitle = taskTitleField.getText();
            String newPriority = newPriorityField.getText();
            updateTask(taskTitle, newPriority, frame, "edit");
        }
    }

    private static void addNotes(JFrame frame) {
        JTextField taskTitleField = new JTextField();
        JTextField notesField = new JTextField();
        Object[] fields = {
                "Task Title to Add Notes:", taskTitleField,
                "Notes:", notesField
        };

        int option = JOptionPane.showConfirmDialog(frame, fields, "Add Notes", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            String taskTitle = taskTitleField.getText();
            String notes = notesField.getText();
            updateTask(taskTitle, notes, frame, "addNotes");
        }
    }

    private static void updateTask(String taskTitle, String updateData, JFrame frame, String mode) {
        File inputFile = new File(TASKS_FILE);
        File tempFile = new File("temp.txt");

        try (BufferedReader reader = new BufferedReader(new FileReader(inputFile));
             BufferedWriter writer = new BufferedWriter(new FileWriter(tempFile))) {

            String line;
            boolean found = false;
            while ((line = reader.readLine()) != null) {
                String[] task = line.split(":");
                if (task[0].equals(currentUser) && task[1].equals(taskTitle)) {
                    if (mode.equals("edit")) {
                        writer.write(currentUser + ":" + taskTitle + ":" + updateData + ":" + (task.length > 3 ? task[3] : ""));
                    } else if (mode.equals("addNotes")) {
                        writer.write(currentUser + ":" + taskTitle + ":" + task[2] + ":" + updateData);
                    }
                    writer.newLine();
                    found = true;
                } else {
                    writer.write(line);
                    writer.newLine();
                }
            }

            if (!found) {
                JOptionPane.showMessageDialog(frame, "Task not found.");
            } else {
                JOptionPane.showMessageDialog(frame, mode.equals("edit") ? "Task updated successfully." : "Notes added successfully.");
            }

        } catch (IOException e) {
            JOptionPane.showMessageDialog(frame, "Error updating task.");
        }

        if (!inputFile.delete() || !tempFile.renameTo(inputFile)) {
            JOptionPane.showMessageDialog(frame, "Error updating tasks file.");
        }
    }
}
