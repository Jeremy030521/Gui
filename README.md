# Gui
//This system provides a user-friendly graphical user interface (GUI) to allow users to easily interact with the scheduling simulator without using command-line operations.

import javafx.application.Application;
import javafx.geometry.Pos;
import javafx.stage.Stage;
import javafx.scene.Scene;
import javafx.scene.control.Label;
import javafx.scene.control.Button;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.layout.GridPane;
import javafx.scene.layout.HBox;
import javafx.scene.layout.VBox;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

public class Lab4 extends Application {
    public static final int SIZE = 8; // The size of the chess board

    // queens[i] indicates the position of the queen in the ith row
    
    private int[] queens = {-1, -1, -1, -1, -1, -1, -1, -1};

    // solution[row] = col
    private List<int[]> solutions =
            Collections.synchronizedList(new ArrayList<>());

    
    private AtomicInteger solutionCount = new AtomicInteger(0);

    // UI 
    private Label[][] labels = new Label[SIZE][SIZE];
    private int currentIndex = 0;       // 当前显示的是第几个解（0-based）
    private Label infoLabel = new Label(); // 显示 "k / n" 1/92
    private Image queenImage;           // 皇后图片（可能为 null）

    @Override // Override the start method in the Application class
    public void start(Stage primaryStage) {
       
        parallelSearch();

        
        try {
            queenImage = new Image("image/queen.jpg");
        } catch (IllegalArgumentException ex) {
            queenImage = null;
            System.out.println("queen.jpg not found, will use text Q instead.");
        }

        //  GridPane（8×8）
        GridPane chessBoard = new GridPane();
        chessBoard.setAlignment(Pos.CENTER);
        // i = 行, j = 列
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                Label cell = new Label();
                cell.setPrefSize(55, 55);
                
                if ((i + j) % 2 == 0)
                    cell.setStyle("-fx-background-color: white; -fx-border-color: black;");
                else
                    cell.setStyle("-fx-background-color: black; -fx-border-color: black;");
                labels[i][j] = cell;
                chessBoard.add(cell, j, i);
            }
        }

        
        Button btnPrev = new Button("Previous Solution");
        Button btnNext = new Button("Next Solution");

        btnPrev.setOnAction(e -> {
            if (!solutions.isEmpty()) {
                currentIndex = (currentIndex - 1 + solutions.size()) % solutions.size();
                showSolution(currentIndex);
            }
        });

        btnNext.setOnAction(e -> {
            if (!solutions.isEmpty()) {
                currentIndex = (currentIndex + 1) % solutions.size();
                showSolution(currentIndex);
            }
        });

        HBox buttons = new HBox(10, btnPrev, btnNext, infoLabel);
        buttons.setAlignment(Pos.CENTER);

        VBox root = new VBox(10, chessBoard, buttons);
        root.setAlignment(Pos.CENTER);

        Scene scene = new Scene(root, 55 * SIZE, 55 * SIZE + 70);
        primaryStage.setTitle("Eight Queens Solutions");
        primaryStage.setScene(scene);
        primaryStage.show();

        
        if (!solutions.isEmpty()) {
            currentIndex = 0;
            showSolution(0);
        }

        System.out.println("Number of solutions = " + solutionCount.get());
    }

    

    /**
     * Parallel Eight Queens:
     
     */
    private void parallelSearch() {
        Thread[] threads = new Thread[SIZE];

        for (int col = 0; col < SIZE; col++) {
            final int firstCol = col;
            threads[col] = new Thread(new QueenTask(firstCol));
            threads[col].start();
        }

        
        for (int i = 0; i < SIZE; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException ex) {
                ex.printStackTrace();
            }
        }
        Collections.sort(solutions, (a, b) -> Integer.compare(a[0], b[0]));
    }

   
    private class QueenTask implements Runnable {
        private int firstCol;

        public QueenTask(int firstCol) {
            this.firstCol = firstCol;
        }

        @Override
        public void run() {
            
            int[] localQueens = new int[SIZE];
            for (int i = 0; i < SIZE; i++)
                localQueens[i] = -1;

            localQueens[0] = firstCol;      // row 0 固定一列
            searchFromRow(localQueens, 1);  // 从 row 1 开始搜索
        }
    }

   
    private void searchFromRow(int[] localQueens, int row) {
        if (row == SIZE) { // 0..7 行都有皇后，找到一组解
            solutionCount.incrementAndGet();
            
            int[] solution = localQueens.clone();
            
            synchronized (solutions) {
                solutions.add(solution);
            }
            return;
        }

        for (int col = 0; col < SIZE; col++) {
            if (isValid(localQueens, row, col)) {
                localQueens[row] = col;
                searchFromRow(localQueens, row + 1);
            }
        }
    }

    
    public boolean isValid(int[] q, int row, int column) {
        for (int i = 1; i <= row; i++)
            if (q[row - i] == column                // Check column
                    || q[row - i] == column - i     // Check upleft diagonal
                    || q[row - i] == column + i)    // Check upright diagonal
                return false;                       // There is a conflict
        return true;                                // No conflict
    }

   

   
    private void showSolution(int index) {
        
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                labels[i][j].setGraphic(null);
                labels[i][j].setText("");
            }
        }

        int[] sol = solutions.get(index);
        for (int row = 0; row < SIZE; row++) {
            queens[row] = sol[row]; 
            int col = sol[row];
            Label cell = labels[row][col];
            if (queenImage != null) {
                cell.setGraphic(new ImageView(queenImage));
            } else {
                
                cell.setText("Q");
                cell.setStyle(cell.getStyle() + "; -fx-text-fill: red; -fx-alignment: center;");
            }
        }

        infoLabel.setText((index + 1) + " / " + solutions.size());
    }

    public static void main(String[] args) {
        Application.launch(args);
    }
}
