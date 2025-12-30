# Banking-System
A System That Takes Information About Customers And Their Money
import javax.swing.JOptionPane;
import java.util.ArrayList;

abstract class Account {
    protected int accountNumber;
    protected String accountName;
    protected double balance;
    protected String nationalId;

    public Account(int accountNumber, String accountName, double balance, String nationalId) {
        this.accountNumber = accountNumber;
        this.accountName = accountName;
        this.balance = balance;
        this.nationalId = nationalId;
    }

    public int getAccountNumber() { return accountNumber; }
    public String getAccountName() { return accountName; }
    public double getBalance() { return balance; }
    public String getNationalId() { return nationalId; }

    public void deposit(double amount) { balance += amount; }
    public abstract boolean withdraw(double amount);
    public abstract String getType();
}

class CheckingAccount extends Account {
    public CheckingAccount(int accountNumber, String accountName, double balance, String nationalId) {
        super(accountNumber, accountName, balance, nationalId);
    }

    @Override
    public boolean withdraw(double amount) {
        if(amount <= balance) { balance -= amount; return true; }
        JOptionPane.showMessageDialog(null, "Error: Insufficient balance");
        return false;
    }

    @Override
    public String getType() { return "Checking"; }
}

class SavingsAccount extends Account {
    private int withdrawalsCount;

    public SavingsAccount(int accountNumber, String accountName, double balance, String nationalId) {
        super(accountNumber, accountName, balance, nationalId);
        this.withdrawalsCount = 0;
    }

    @Override
    public boolean withdraw(double amount) {
        if(withdrawalsCount >= 3) { JOptionPane.showMessageDialog(null, "Error: Withdrawal limit reached"); return false; }
        if(balance - amount < 100) { JOptionPane.showMessageDialog(null, "Error: Savings account cannot drop below $100"); return false; }
        balance -= amount;
        withdrawalsCount++;
        return true;
    }

    public void applyInterest() { balance += balance * 0.02; JOptionPane.showMessageDialog(null, "Interest applied to Account #" + accountNumber); }

    @Override
    public String getType() { return "Savings"; }
}

class Bank {
    private ArrayList<Account> accounts = new ArrayList<>();
    private int nextAccountNumber = 1; // يبدأ من 1

    public void addAccount(Account acc) {
        accounts.add(acc);
        if(acc.getAccountNumber() >= nextAccountNumber) nextAccountNumber = acc.getAccountNumber() + 1;
    }

    private String getNameInput(String message) {
        String input = "";
        do {
            input = JOptionPane.showInputDialog(message);
            if(input == null || input.trim().isEmpty()) { JOptionPane.showMessageDialog(null, "Input cannot be empty!"); input = ""; }
            else if(!input.matches("[a-zA-Z ]+")) { JOptionPane.showMessageDialog(null, "Invalid input! Use English letters only."); input = ""; }
        } while(input.isEmpty());
        return input.trim();
    }

    private int getIntInput(String message) {
        while(true) {
            try { String input = JOptionPane.showInputDialog(message); return Integer.parseInt(input); }
            catch(NumberFormatException e) { JOptionPane.showMessageDialog(null, "Invalid input! Please enter a number."); }
        }
    }

    private double getDoubleInput(String message) {
        while(true) {
            try { String input = JOptionPane.showInputDialog(message); return Double.parseDouble(input); }
            catch(NumberFormatException e) { JOptionPane.showMessageDialog(null, "Invalid input! Please enter a valid amount."); }
        }
    }

    private String getNationalIdInput(String message) {
        String input = "";
        do {
            input = JOptionPane.showInputDialog(message);
            if(input == null || input.trim().isEmpty()) { JOptionPane.showMessageDialog(null, "Input cannot be empty!"); input = ""; }
            else if(!input.matches("\\d{14}")) { JOptionPane.showMessageDialog(null, "Invalid National ID! Must be 14 digits."); input = ""; }
        } while(input.isEmpty());
        return input.trim();
    }

    public Account findAccount(int accNo) {
        for(Account acc : accounts) if(acc.getAccountNumber() == accNo) return acc;
        return null;
    }

    public Account findAccountByNationalId(String nationalId) {
        for(Account acc : accounts) if(acc.getNationalId().equals(nationalId)) return acc;
        return null;
    }

    public void createAccount() {
        String name = getNameInput("Enter Account Name (English letters only):");
        String nationalId = getNationalIdInput("Enter your National ID (14 digits):");
        String[] types = {"Checking", "Savings"};
        int typeChoice = JOptionPane.showOptionDialog(null, "Select Account Type:", "Account Type", JOptionPane.DEFAULT_OPTION, JOptionPane.PLAIN_MESSAGE, null, types, types[0]);
        if(typeChoice == -1) return;
        double balance = getDoubleInput("Enter Initial Deposit:");
        if(typeChoice == 1 && balance < 100) { JOptionPane.showMessageDialog(null, "Savings account requires minimum $100"); return; }
        Account acc = (typeChoice == 0) ? new CheckingAccount(nextAccountNumber, name, balance, nationalId)
                : new SavingsAccount(nextAccountNumber, name, balance, nationalId);
        addAccount(acc);
        JOptionPane.showMessageDialog(null, "Account Created Successfully!\nYour Account ID is: " + nextAccountNumber);
        nextAccountNumber++;
    }

    private Account getAccountForTransaction(String prefix) {
        int accNo = getIntInput("Enter " + prefix + " Account Number:");
        Account acc = findAccount(accNo);
        if(acc == null) {
            JOptionPane.showMessageDialog(null, "Account not found! Use National ID instead.");
            String nid = getNationalIdInput("Enter " + prefix + " National ID:");
            acc = findAccountByNationalId(nid);
            if(acc == null) { JOptionPane.showMessageDialog(null, "No account found for this National ID."); }
        }
        return acc;
    }

    public void deposit() {
        Account acc = getAccountForTransaction("");
        if(acc == null) return;
        double amount = getDoubleInput("Enter Amount to Deposit:");
        acc.deposit(amount);
        JOptionPane.showMessageDialog(null, "Deposited $" + amount);
    }

    public void withdraw() {
        Account acc = getAccountForTransaction("");
        if(acc == null) return;
        double amount = getDoubleInput("Enter Amount to Withdraw:");
        acc.withdraw(amount);
    }

    public void transfer() {
        Account src = getAccountForTransaction("Source");
        if(src == null) return;
        Account dest = getAccountForTransaction("Destination");
        if(dest == null) return;
        double amount = getDoubleInput("Enter Amount to Transfer:");
        if(src.withdraw(amount)) {
            dest.deposit(amount);
            JOptionPane.showMessageDialog(null, "Transferred $" + amount + " to Account #" + dest.getAccountNumber());
        }
    }

    public void applyInterest() {
        for(Account acc : accounts) if(acc instanceof SavingsAccount) ((SavingsAccount)acc).applyInterest();
    }

    public void checkAccount() {
        Account acc = null;
        int attempts = 0;
        while(attempts < 3 && acc == null) {
            int accNo = getIntInput("Enter Account Number:");
            acc = findAccount(accNo);
            attempts++;
        }
        if(acc == null) {
            String nid = getNationalIdInput("Enter National ID to access your account:");
            acc = findAccountByNationalId(nid);
            if(acc == null) { JOptionPane.showMessageDialog(null, "No account found for this National ID."); return; }
        }
        String info = "Account Number: " + acc.getAccountNumber() +
                "\nAccount Name: " + acc.getAccountName() +
                "\nAccount Type: " + acc.getType() +
                "\nBalance: $" + String.format("%.2f", acc.getBalance());
        JOptionPane.showMessageDialog(null, info, "Account Info", JOptionPane.INFORMATION_MESSAGE);
    }
}

public class main {
    public static void main(String[] args) {
        Bank bank = new Bank();
        String[] options = {"Create Account", "Deposit", "Withdraw", "Transfer", "Apply Interest", "Check Account", "Exit"};
        while(true) {
            int choice = JOptionPane.showOptionDialog(null, "<html><h2>Bank System</h2>Select an option:</html>", "Bank Menu", JOptionPane.DEFAULT_OPTION, JOptionPane.PLAIN_MESSAGE, null, options, options[0]);
            if(choice == -1 || choice == 6) System.exit(0);
            switch(choice) {
                case 0: bank.createAccount(); break;
                case 1: bank.deposit(); break;
                case 2: bank.withdraw(); break;
                case 3: bank.transfer(); break;
                case 4: bank.applyInterest(); break;
                case 5: bank.checkAccount(); break;
            }
        }
    }
}

