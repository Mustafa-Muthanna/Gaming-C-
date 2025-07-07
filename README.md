# Gaming-C-


#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include <SFML/System.hpp>  
#include <iostream>
#include <vector>
#include <string>
#include <ctime>
#include <cstdlib>
#include <random>
#include <sstream>

using namespace sf;
using namespace std;


const Color backgroundColor(30, 30, 45);       // Dark space blue
const Color buttonColor(67, 97, 238);          // Vibrant blue
const Color hoverColor(100, 255, 218);         // Cyan hover effect
const Color textColor(245, 245, 247);          // Off-white text
const Color scoreColor(46, 213, 115);          // Emerald green
const Color warningColor(255, 71, 87);         // Coral red
const Color accentColor(255, 193, 7);          // Gold accents
const Color secondaryColor(163, 93, 255);      // Purple secondary
struct GameResult {
    string playerName;
    int wins;
    int losses;
    int draws;
};
void drawGrid(RenderWindow& window) {
    VertexArray grid(Lines, 8);
    grid[0].position = Vector2f(300, 100);
    grid[1].position = Vector2f(300, 400);
    grid[2].position = Vector2f(500, 100);
    grid[3].position = Vector2f(500, 400);
    grid[4].position = Vector2f(100, 200);
    grid[5].position = Vector2f(700, 200);
    grid[6].position = Vector2f(100, 300);
    grid[7].position = Vector2f(700, 300);
    for (int i = 0; i < 8; i++)
        grid[i].color = Color(200, 200, 220, 150);
    window.draw(grid);
}
class Button {
public:
    RectangleShape shape;
    Text text;

    Button(const string& str, Font& font, Vector2f pos, Vector2f size = Vector2f(300, 50)) {
        shape.setSize(size);
        shape.setFillColor(secondaryColor);
        shape.setOutlineThickness(2);
        shape.setOutlineColor(accentColor);
        shape.setPosition(pos);

        text.setString(str);
        text.setFont(font);
        text.setCharacterSize(28);
        text.setFillColor(textColor);
        FloatRect bounds = text.getLocalBounds();
        text.setPosition(pos.x + (size.x - bounds.width) / 2.f - bounds.left,
            pos.y + (size.y - bounds.height) / 2.f - bounds.top);
    }

    bool isMouseOver(RenderWindow& window) {
        return shape.getGlobalBounds().contains(window.mapPixelToCoords(Mouse::getPosition(window)));
    }

    void draw(RenderWindow& window) {
        window.draw(shape);
        window.draw(text);
    }
};

class GuessingGame {
    Font& font;
    int answer, guesses, low, high;
    string inputText;
    Text infoText, guessCounter, rangeText;
    RectangleShape inputBox;

public:
    GuessingGame(Font& f) : font(f), low(1), high(100), guesses(0) {
        random_device rd;
        mt19937 gen(rd());
        uniform_int_distribution<> distrib(low, high);
        answer = distrib(gen);

        inputBox.setSize(Vector2f(200, 40));
        inputBox.setFillColor(Color(255, 255, 255, 200));
        inputBox.setOutlineThickness(2);
        inputBox.setOutlineColor(accentColor);
        inputBox.setPosition(300, 200);

        infoText.setFont(font);
        infoText.setCharacterSize(24);
        infoText.setFillColor(textColor);
        infoText.setPosition(250, 150);

        guessCounter.setFont(font);
        guessCounter.setCharacterSize(20);
        guessCounter.setFillColor(textColor);
        guessCounter.setPosition(350, 300);

        rangeText.setFont(font);
        rangeText.setCharacterSize(20);
        rangeText.setFillColor(textColor);
        rangeText.setPosition(300, 350);
        updateRangeText();
    }

    void updateRangeText() {
        rangeText.setString("Range: " + to_string(low) + " - " + to_string(high));
    }

    void StartGame(RenderWindow& window) {
        Text inputDisplay;
        inputDisplay.setFont(font);
        inputDisplay.setCharacterSize(24);
        inputDisplay.setFillColor(Color::Black);
        inputDisplay.setPosition(305, 205);

        Button submitBtn("Submit Guess", font, Vector2f(300, 400));
        Button quitBtn("Quit", font, Vector2f(300, 480));

        infoText.setString("I'm Thinking About a Number");
        infoText.setFillColor(buttonColor);

        while (window.isOpen()) {
            Event event;
            while (window.pollEvent(event)) {
                if (event.type == Event::Closed)
                    window.close();
                if (event.type == Event::TextEntered) {
                    if (event.text.unicode == '\b' && !inputText.empty())
                        inputText.pop_back();
                    else if (event.text.unicode >= '0' && event.text.unicode <= '9')
                        inputText += static_cast<char>(event.text.unicode);
                }
                if (event.type == Event::MouseButtonReleased) {
                    if (submitBtn.isMouseOver(window)) {
                        try {
                            int guess = stoi(inputText);
                            guesses++;
                            if (guess == -1) {
                                infoText.setString("Ha looser!");
                                infoText.setFillColor(warningColor);
                            }
                            else if (guess < low || guess > high) {
                                infoText.setString("Out of range! Try again!");
                                infoText.setFillColor(warningColor);
                            }
                            else if (guess > answer) {
                                infoText.setString("Too high! Try again!");
                                infoText.setFillColor(buttonColor);
                                high = guess - 1;
                                updateRangeText();
                            }
                            else if (guess < answer) {
                                infoText.setString("Too low! Try again!");
                                infoText.setFillColor(buttonColor);
                                low = guess + 1;
                                updateRangeText();
                            }
                            else {
                                infoText.setString("🎉 Correct! You won in " + to_string(guesses) + " guesses!");
                                infoText.setFillColor(scoreColor);
                            }
                            guessCounter.setString("Guesses: " + to_string(guesses));
                            inputText.clear();
                        }
                        catch (...) {
                            infoText.setString("Invalid number!");
                            infoText.setFillColor(warningColor);
                        }
                    }
                    else if (quitBtn.isMouseOver(window)) {
                        return;
                    }
                }
            }
            inputDisplay.setString(inputText);
            submitBtn.shape.setFillColor(submitBtn.isMouseOver(window) ? hoverColor : secondaryColor);
            quitBtn.shape.setFillColor(quitBtn.isMouseOver(window) ? hoverColor : warningColor);

            window.clear(backgroundColor);
            window.draw(inputBox);
            window.draw(inputDisplay);
            window.draw(infoText);
            window.draw(guessCounter);
            window.draw(rangeText);
            submitBtn.draw(window);
            quitBtn.draw(window);
            window.display();
        }
    }
};
class R_P_S {
    Font& font;
    Text resultText;
    RectangleShape vsShape;
    mt19937 rng;
    int ReadNumberOfRounds(RenderWindow& window) {
        Vector2f buttonSize(120, 50);
        float windowWidth = window.getSize().x;
        Vector2f gridStart((windowWidth - 5 * 130) / 2.f, 200);
        vector<Button> roundButtons;
        for (int i = 0; i < 10; i++) {
            Vector2f pos(gridStart.x + (i % 5) * 130, gridStart.y + (i / 5) * 80);
            roundButtons.push_back(Button(to_string(i + 1), font, pos, buttonSize));
        }
        Text prompt("Select Rounds (1-10):", font, 36);
        prompt.setPosition(windowWidth / 2 - prompt.getLocalBounds().width / 2, 100);
        prompt.setFillColor(accentColor);
        int selectedRounds = 0;
        while (window.isOpen() && selectedRounds == 0) {
            Event event;
            while (window.pollEvent(event)) {
                if (event.type == Event::Closed)
                    window.close();
                if (event.type == Event::MouseButtonReleased) {
                    for (int i = 0; i < roundButtons.size(); i++) {
                        if (roundButtons[i].isMouseOver(window))
                            selectedRounds = i + 1;
                    }
                }
            }
            for (auto& btn : roundButtons)
                btn.shape.setFillColor(btn.isMouseOver(window) ? hoverColor : secondaryColor);
            window.clear(backgroundColor);
            window.draw(prompt);
            for (auto& btn : roundButtons)
                btn.draw(window);
            window.display();
        }
        return selectedRounds;
    }

    int IsPlayer1Win(int p1, int comp) {
        if (p1 == comp) return 0;
        if ((p1 == 1 && comp == 3) || (p1 == 2 && comp == 1) || (p1 == 3 && comp == 2))
            return 1;
        return -1;
    }

    void drawChoices(RenderWindow& window, int player, int computer) {
        float centerX = window.getSize().x / 2.f;
        float yPos = 250.f, padding = 180.f;
        CircleShape playerShape;
        playerShape.setOutlineThickness(3);
        playerShape.setOutlineColor(accentColor);
        float pRadius = (player == 1) ? 60.f : (player == 2 ? 45.f : 35.f);
        float playerX = centerX - padding - pRadius;
        playerShape.setRadius(pRadius);
        playerShape.setPosition(playerX, yPos);
        playerShape.setFillColor(buttonColor);
        window.draw(playerShape);

        CircleShape compShape;
        compShape.setOutlineThickness(3);
        compShape.setOutlineColor(warningColor);
        float cRadius = (computer == 1) ? 60.f : (computer == 2 ? 45.f : 35.f);
        float compX = centerX + padding - cRadius;
        compShape.setRadius(cRadius);
        compShape.setPosition(compX, yPos);
        compShape.setFillColor(secondaryColor);
        window.draw(compShape);

        Text vsText("VS", font, 48);
        vsText.setPosition(centerX - vsText.getLocalBounds().width / 2, yPos + 50);
        vsText.setFillColor(accentColor);
        window.draw(vsText);

        auto createLabel = [&](const string& str, float x, float radius) {
            Text label(str, font, 24);
            label.setPosition(x + radius - label.getLocalBounds().width / 2, yPos + radius * 2 + 10);
            label.setFillColor(textColor);
            return label;
            };
        window.draw(createLabel("Player", playerX, pRadius));
        window.draw(createLabel("Computer", compX, cRadius));
    }

    void PlayRound(RenderWindow& window, GameResult& result, int currentRound, int totalRounds) {
        float winWidth = window.getSize().x;
        Vector2f posArr[] = {
            Vector2f((winWidth - 560) / 2, 400),
            Vector2f((winWidth - 120) / 2, 400),
            Vector2f((winWidth - 560) / 2 + 440, 400)
        };
        vector<Button> buttons = {
            Button("Rock", font, posArr[0]),
            Button("Paper", font, posArr[1]),
            Button("Scissors", font, posArr[2])
        };
        int playerChoice = -1;
        while (window.isOpen() && playerChoice == -1) {
            Event event;
            while (window.pollEvent(event)) {
                if (event.type == Event::Closed) window.close();
                if (event.type == Event::MouseButtonReleased) {
                    for (int i = 0; i < buttons.size(); i++) {
                        if (buttons[i].isMouseOver(window))
                            playerChoice = i + 1;
                    }
                }
            }
            for (auto& btn : buttons)
                btn.shape.setFillColor(btn.isMouseOver(window) ? hoverColor : secondaryColor);
            window.clear(backgroundColor);
            Text roundText("Choose your weapon!", font, 36);
            roundText.setPosition(winWidth / 2 - roundText.getLocalBounds().width / 2, 50);
            Text roundNum("Round " + to_string(currentRound) + "/" + to_string(totalRounds), font, 24);
            roundNum.setPosition(winWidth / 2 - roundNum.getLocalBounds().width / 2, 100);
            window.draw(roundText);
            window.draw(roundNum);
            for (auto& btn : buttons)
                btn.draw(window);
            window.display();
        }
        uniform_int_distribution<> distrib(1, 3);
        int computerChoice = distrib(rng);
        int roundResult = IsPlayer1Win(playerChoice, computerChoice);
        result.draws += (roundResult == 0);
        result.wins += (roundResult == 1);
        result.losses += (roundResult == -1);
        window.clear(backgroundColor);
        drawChoices(window, playerChoice, computerChoice);
        Text resText;
        resText.setFont(font);
        resText.setCharacterSize(36);
        if (roundResult == 0) {
            resText.setString("Draw!");
            resText.setFillColor(textColor);
        }
        else if (roundResult == 1) {
            resText.setString("You Win!");
            resText.setFillColor(scoreColor);
        }
        else {
            resText.setString("Computer Wins!");
            resText.setFillColor(warningColor);
        }
        FloatRect tb = resText.getLocalBounds();
        resText.setOrigin(tb.left + tb.width / 2.f, tb.top + tb.height / 2.f);
        resText.setPosition(window.getSize().x / 2.f, 400);
        window.draw(resText);
        window.display();
        sf::sleep(sf::milliseconds(2000));
    }

public:
    R_P_S(Font& f) : font(f), rng(random_device{}()) {
        vsShape.setSize(Vector2f(100, 100));
        vsShape.setFillColor(textColor);
    }

    void StartGame(RenderWindow& window) {
        GameResult result{ "Player1", 0, 0, 0 };
        int totalRounds = ReadNumberOfRounds(window);
        if (totalRounds <= 0)
            return;
        for (int currRound = 1; currRound <= totalRounds; currRound++)
            PlayRound(window, result, currRound, totalRounds);
        window.clear(backgroundColor);
        float wWidth = window.getSize().x;
        Text gameOver("Game Over!", font, 48);
        gameOver.setPosition(wWidth / 2 - gameOver.getLocalBounds().width / 2, 50);
        Text stats("Wins: " + to_string(result.wins) + "\nLosses: " + to_string(result.losses) +
            "\nDraws: " + to_string(result.draws), font, 36);
        FloatRect tbFinal = stats.getLocalBounds();
        stats.setOrigin(tbFinal.left + tbFinal.width / 2.f, tbFinal.top + tbFinal.height / 2.f);
        stats.setPosition(wWidth / 2.f, 300);
        window.draw(gameOver);
        window.draw(stats);
        window.display();
        sf::sleep(sf::milliseconds(3000));
    }
};

class TickTakToe {
    Font& font;
    char board[3][3];
    char currentPlayer;
public:
    TickTakToe(Font& f) : font(f), currentPlayer('X') {
        char num = '1';
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                board[i][j] = num++;
    }

    bool CheckWin(char player) {
        for (int i = 0; i < 3; i++)
            if (board[i][0] == player && board[i][1] == player && board[i][2] == player)
                return true;
        for (int j = 0; j < 3; j++)
            if (board[0][j] == player && board[1][j] == player && board[2][j] == player)
                return true;
        if (board[0][0] == player && board[1][1] == player && board[2][2] == player)
            return true;
        if (board[0][2] == player && board[1][1] == player && board[2][0] == player)
            return true;
        return false;
    }

    bool CheckDraw() {
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                if (board[i][j] != 'X' && board[i][j] != 'O')
                    return false;
        return true;
    }

    void drawBoard(RenderWindow& window) {
        window.clear(backgroundColor);
        drawGrid(window);
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (board[i][j] == 'X' || board[i][j] == 'O') {
                    Text symbol(string(1, board[i][j]), font, 48);
                    symbol.setPosition(200 + j * 200 - 25, 150 + i * 100 - 40);
                    symbol.setFillColor(board[i][j] == 'X' ? hoverColor : secondaryColor);
                    window.draw(symbol);
                }
            }
        }
    }

    void StartGame(RenderWindow& window) {
        bool gameOver = false;
        char winner = '\0';
        bool draw = false;
        Event event;
        while (window.isOpen() && !gameOver) {
            while (window.pollEvent(event)) {
                if (event.type == Event::Closed)
                    window.close();
                if (event.type == Event::MouseButtonReleased) {
                    Vector2i mousePos = Mouse::getPosition(window);
                    int col = (mousePos.x - 100) / 200;
                    int row = (mousePos.y - 100) / 100;
                    if (row >= 0 && row < 3 && col >= 0 && col < 3 &&
                        board[row][col] != 'X' && board[row][col] != 'O') {
                        board[row][col] = currentPlayer;
                        if (CheckWin(currentPlayer)) {
                            winner = currentPlayer;
                            gameOver = true;
                        }
                        else if (CheckDraw()) {
                            draw = true;
                            gameOver = true;
                        }
                        else {
                            currentPlayer = (currentPlayer == 'X') ? 'O' : 'X';
                        }
                    }
                }
            }
            drawBoard(window);
            window.display();
        }
        if (gameOver) {
            window.clear(backgroundColor);
            drawBoard(window);
            Text resultText;
            resultText.setFont(font);
            resultText.setCharacterSize(48);
            if (winner != '\0') {
                resultText.setString("Player " + string(1, winner) + " wins!");
                resultText.setFillColor(winner == 'X' ? hoverColor : secondaryColor);
            }
            else if (draw) {
                resultText.setString("It's a draw!");
                resultText.setFillColor(scoreColor);
            }
            FloatRect tb = resultText.getLocalBounds();
            resultText.setOrigin(tb.left + tb.width / 2.f, tb.top + tb.height / 2.f);
            resultText.setPosition(window.getSize().x / 2.f, 450);
            window.draw(resultText);
            window.display();
            sf::sleep(sf::seconds(3));
        }
    }
};
class UltimateTicTacToe {
    Font& font;
    char board[3][3];
    char currentPlayer;
    int playerYCount;
public:
    UltimateTicTacToe(Font& f) : font(f), currentPlayer('X'), playerYCount(0) {
        char num = '1';
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                board[i][j] = num++;
    }

    bool CheckWin(char player) {
        if (player == 'Y' && playerYCount >= 3)
            return true;
        for (int i = 0; i < 3; i++)
            if (board[i][0] == player && board[i][1] == player && board[i][2] == player)
                return true;
        for (int j = 0; j < 3; j++)
            if (board[0][j] == player && board[1][j] == player && board[2][j] == player)
                return true;
        if (board[0][0] == player && board[1][1] == player && board[2][2] == player)
            return true;
        if (board[0][2] == player && board[1][1] == player && board[2][0] == player)
            return true;
        return false;
    }

    bool CheckDraw() {
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                if (board[i][j] != 'X' && board[i][j] != 'Y')
                    return false;
        return true;
    }

    void drawBoard(RenderWindow& window) {
        window.clear(backgroundColor);
        drawGrid(window);
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (board[i][j] == 'X' || board[i][j] == 'Y') {
                    Text symbol(string(1, board[i][j]), font, 48);
                    symbol.setPosition(200 + j * 200 - 25, 150 + i * 100 - 40);
                    symbol.setFillColor(board[i][j] == 'X' ? hoverColor : secondaryColor);
                    window.draw(symbol);
                }
            }
        }
    }

    void StartGame(RenderWindow& window) {
        bool gameOver = false;
        char winner = '\0';
        bool draw = false;
        Event event;
        while (window.isOpen() && !gameOver) {
            while (window.pollEvent(event)) {
                if (event.type == Event::Closed)
                    window.close();
                if (event.type == Event::MouseButtonReleased) {
                    Vector2i mousePos = Mouse::getPosition(window);
                    int col = (mousePos.x - 100) / 200;
                    int row = (mousePos.y - 100) / 100;
                    if (row >= 0 && row < 3 && col >= 0 && col < 3 &&
                        board[row][col] != 'X' && board[row][col] != 'Y') {
                        board[row][col] = currentPlayer;
                        if (currentPlayer == 'Y')
                            playerYCount++;
                        if (CheckWin(currentPlayer)) {
                            winner = currentPlayer;
                            gameOver = true;
                        }
                        else if (CheckDraw()) {
                            draw = true;
                            gameOver = true;
                        }
                        else {
                            currentPlayer = (currentPlayer == 'X') ? 'Y' : 'X';
                        }
                    }
                }
            }
            drawBoard(window);
            window.display();
        }
        window.clear(backgroundColor);
        drawBoard(window);
        Text resultText;
        resultText.setFont(font);
        resultText.setCharacterSize(48);
        if (winner != '\0') {
            if (winner == 'X')
                winner = 'Y';
            resultText.setString("Player " + string(1, winner) + " wins!");
            resultText.setFillColor(winner == 'X' ? hoverColor : secondaryColor);
        }
        else if (draw) {
            resultText.setString("It's a draw!");
            resultText.setFillColor(scoreColor);
        }
        FloatRect tb = resultText.getLocalBounds();
        resultText.setOrigin(tb.left + tb.width / 2.f, tb.top + tb.height / 2.f);
        resultText.setPosition(window.getSize().x / 2.f, 450);
        window.draw(resultText);
        window.display();
        sf::sleep(sf::seconds(3));
        currentPlayer = 'X';
        playerYCount = 0;
        char num = '1';
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                board[i][j] = num++;
    }
};

class Player {
public:
    Player() {
        shape.setSize(sf::Vector2f(50, 50));
        shape.setFillColor(sf::Color(100, 250, 50));
        shape.setPosition(375, 500);
    }

    void move(const sf::Keyboard::Key& key) {
        if (key == sf::Keyboard::Left && shape.getPosition().x > 0)
            shape.move(-5, 0); 
        if (key == sf::Keyboard::Right && shape.getPosition().x < 750)
            shape.move(5, 0);
    }

    const sf::RectangleShape& getShape() const {
        return shape;
    }

private:
    sf::RectangleShape shape;
};

class Obstacle {
public:
    Obstacle(float speed, float xPos) {
        shape.setRadius(20);
        shape.setFillColor(sf::Color(rand() % 256, rand() % 256, rand() % 256)); // Randomized color
        shape.setOutlineColor(sf::Color::Black); // Added border
        shape.setOutlineThickness(2);
        shape.setPosition(xPos, 0);
        this->speed = speed;
    }

    void move() {
        shape.move(0, speed);
    }

    bool isOffScreen() const {
        return shape.getPosition().y > 600;
    }

    const sf::CircleShape& getShape() const {
        return shape;
    }

private:
    sf::CircleShape shape;
    float speed;
};

class Game {
public:
    Game()
        : window(sf::VideoMode(800, 600), "Avoid the Obstacles!"), score(0), spawnTimer(0),
        spawnTimerThreshold(60), gameOver(false) {
        window.setFramerateLimit(60);
        font.loadFromFile("arial.ttf");
        scoreText.setFont(font);
        scoreText.setCharacterSize(24);
        scoreText.setFillColor(sf::Color::White); // Enhanced visibility
        scoreText.setPosition(10, 10);
        bgColor = sf::Color(50, 50, 150); // Soft gradient background
    }

    void run() {
        while (window.isOpen()) {
            handleEvents();
            if (!gameOver) {
                update();
                
            }
            render();
           
        }
       
    }

private:
    sf::RenderWindow window;
    Player player;
    std::vector<Obstacle> obstacles;
    int score;
    int spawnTimer;
    int spawnTimerThreshold;
    bool gameOver;
    sf::Font font;
    sf::Text scoreText;
    sf::Color bgColor;

    void handleEvents() {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();
           

            if (event.type == sf::Event::KeyPressed) {
                player.move(event.key.code);
            }
        }
    }

    void update() {
        spawnTimerThreshold = std::max(60 - (score / 500) * 5, 30);
        spawnTimer++;
        if (spawnTimer >= spawnTimerThreshold) {
            float xPos = rand() % 700;
            float speed = (rand() % 3) + 5 + (score / 500);
            obstacles.emplace_back(speed, xPos);
            spawnTimer = 0;
        }

        for (auto it = obstacles.begin(); it != obstacles.end();) {
            it->move();
            if (it->getShape().getGlobalBounds().intersects(player.getShape().getGlobalBounds())) {
                gameOver = true;
                window.close(); 
                return;


            }
            if (it->isOffScreen()) {
                it = obstacles.erase(it);
                score += 10;
                
            }
            else {
                ++it;
                
            }
            
        }
        
        bgColor.r = (bgColor.r + 1) % 256;
        bgColor.b = (bgColor.b + 2) % 256;

        scoreText.setString("Score: " + std::to_string(score));
        
    }

    void render() {
        window.clear(bgColor);
        window.draw(player.getShape());
        for (const auto& obstacle : obstacles) {
            window.draw(obstacle.getShape());
        }
        window.draw(scoreText);

        if (gameOver) {
            sf::Text gameOverText;
            gameOverText.setFont(font);
            gameOverText.setCharacterSize(50);
            gameOverText.setFillColor(sf::Color::Red);
            gameOverText.setOutlineColor(sf::Color::Black);
            gameOverText.setOutlineThickness(2);
            gameOverText.setString("Game Over! Final Score: " + std::to_string(score));
            gameOverText.setPosition(100, 250);
            window.draw(gameOverText);
        }

        window.display();
    }
};

class SnakeGame {
public:
    SnakeGame()
        : window(sf::VideoMode(800, 600), "Snake Game"), direction(0, -1), blockSize(20), gameOver(false) {
        window.setFramerateLimit(10);

        // Initialize the snake
        for (int i = 0; i < 3; ++i) {
            sf::RectangleShape block(sf::Vector2f(blockSize, blockSize));
            block.setFillColor(sf::Color(0, 255 - i * 30, 0));
            block.setPosition(400, 300 + i * blockSize);
            snake.push_back(block);
        }

        // Initialize the food
        food.setSize(sf::Vector2f(blockSize, blockSize));
        food.setFillColor(sf::Color::Red);
        spawnFood();

        // Add obstacles and gardens
        addObstaclesAndGardens();
    }

    void run() {
        while (window.isOpen()) {
            handleEvents();
            if (!gameOver) {
                update();
            }
            else {
                window.close();
            }
            render();
        }
    }

private:
    sf::RenderWindow window;
    std::vector<sf::RectangleShape> snake;
    sf::RectangleShape food;
    sf::Vector2i direction;
    const int blockSize;
    bool gameOver;
    sf::Clock clock;
    std::vector<sf::RectangleShape> obstacles;

    void handleEvents() {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed) {
                window.close();
            }
        }

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up) && direction != sf::Vector2i(0, 1)) {
            direction = sf::Vector2i(0, -1);
        }
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down) && direction != sf::Vector2i(0, -1)) {
            direction = sf::Vector2i(0, 1);
        }
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left) && direction != sf::Vector2i(1, 0)) {
            direction = sf::Vector2i(-1, 0);
        }
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right) && direction != sf::Vector2i(-1, 0)) {
            direction = sf::Vector2i(1, 0);
        }
    }

    void update() {
        for (int i = snake.size() - 1; i > 0; --i) {
            snake[i].setPosition(snake[i - 1].getPosition());
        }
        snake[0].move(direction.x * blockSize, direction.y * blockSize);

        if (snake[0].getGlobalBounds().intersects(food.getGlobalBounds())) {
            sf::RectangleShape newBlock(sf::Vector2f(blockSize, blockSize));
            newBlock.setFillColor(sf::Color(0, 255 - snake.size() * 5, 0));
            newBlock.setPosition(snake.back().getPosition());
            snake.push_back(newBlock);
            spawnFood();
        }

        if (snake[0].getPosition().x < 0 || snake[0].getPosition().x >= 800 ||
            snake[0].getPosition().y < 0 || snake[0].getPosition().y >= 600) {
            gameOver = true;
        }

        for (size_t i = 1; i < snake.size(); ++i) {
            if (snake[0].getPosition() == snake[i].getPosition()) {
                gameOver = true;
            }
        }

        for (const auto& obstacle : obstacles) {
            if (snake[0].getGlobalBounds().intersects(obstacle.getGlobalBounds())) {
                gameOver = true;
            }
        }
    }

    void render() {
        // Draw dynamic gradient background
        sf::RectangleShape gradient(sf::Vector2f(window.getSize().x, window.getSize().y));
        gradient.setFillColor(sf::Color(
            50 + 20 * std::sin(clock.getElapsedTime().asSeconds()),
            50 + 20 * std::sin(clock.getElapsedTime().asSeconds() + 2),
            70 + 20 * std::sin(clock.getElapsedTime().asSeconds() + 4)
        ));
        window.clear();
        window.draw(gradient);

        // Draw the snake
        for (const auto& block : snake) {
            sf::RectangleShape snakeBlock(block);
            snakeBlock.setOutlineThickness(2);
            snakeBlock.setOutlineColor(sf::Color(0, 128, 128));
            window.draw(snakeBlock);
        }

        // Add glowing effect to the food
        float scale = 1 + 0.1f * std::sin(clock.getElapsedTime().asSeconds() * 5);
        food.setScale(scale, scale);
        window.draw(food);

        // Draw gardens and obstacles
        for (const auto& obstacle : obstacles) {
            window.draw(obstacle);
        }

        // Game Over screen
        if (gameOver) {
            sf::Font font;
            if (!font.loadFromFile("arial.ttf")) {
                return;
            }

            sf::Text gameOverText;
            gameOverText.setFont(font);
            gameOverText.setString("Game Over!");
            gameOverText.setCharacterSize(50);
            gameOverText.setFillColor(sf::Color::Red);
            gameOverText.setOutlineColor(sf::Color::Black);
            gameOverText.setOutlineThickness(5);
            gameOverText.setPosition(250, 250);

            window.draw(gameOverText);
        }

        window.display();
    }

    void spawnFood() {
        int gridWidth = window.getSize().x / blockSize;
        int gridHeight = window.getSize().y / blockSize;

        int foodX = (std::rand() % gridWidth) * blockSize;
        int foodY = (std::rand() % gridHeight) * blockSize;

        food.setPosition(foodX, foodY);
        clock.restart();
    }

    void addObstaclesAndGardens() {
        sf::RectangleShape garden(sf::Vector2f(blockSize * 5, blockSize * 5));
        garden.setFillColor(sf::Color(50, 200, 50));
        garden.setPosition(100, 100);
        obstacles.push_back(garden);

        for (int i = 0; i < 10; ++i) {
            sf::RectangleShape obstacle(sf::Vector2f(blockSize, blockSize));
            obstacle.setFillColor(sf::Color(150, 75, 0));
            obstacle.setPosition(i * blockSize, 0);
            obstacles.push_back(obstacle);

            obstacle.setPosition(i * blockSize, 600 - blockSize);
            obstacles.push_back(obstacle);
        }

        for (int i = 0; i < 10; ++i) {
            sf::RectangleShape obstacle(sf::Vector2f(blockSize, blockSize));
            obstacle.setFillColor(sf::Color(150, 75, 0));
            obstacle.setPosition(0, i * blockSize);
            obstacles.push_back(obstacle);

            obstacle.setPosition(800 - blockSize, i * blockSize);
            obstacles.push_back(obstacle);
        }
    }
};


class GameMenu {
    sf::Font& font;
    std::vector<Button> buttons;
    sf::RectangleShape backgroundGradient;
    sf::RectangleShape titleBox;

public:
    GameMenu(sf::Font& f) : font(f) {
        backgroundGradient.setSize(sf::Vector2f(800, 800));
        backgroundGradient.setFillColor(sf::Color(20, 20, 35));
        titleBox.setSize(sf::Vector2f(600, 80));
        titleBox.setPosition(100, 20);
        titleBox.setFillColor(sf::Color(255, 255, 255, 15));
        titleBox.setOutlineThickness(2);
        titleBox.setOutlineColor(accentColor);

        buttons = {
            Button("Snake Game", font, sf::Vector2f(250, 130)), 
            Button("Avoid the Obstacles", font, sf::Vector2f(250, 230)),
            Button("Rock Paper Scissors", font, sf::Vector2f(250, 330)),
            Button("Ultimate Tic Tac Toe", font, sf::Vector2f(250, 430)),
            Button("Tic Tac Toe", font, sf::Vector2f(250, 530)),
            Button("Guessing Game", font, sf::Vector2f(250, 630)),
            Button("Exit", font, sf::Vector2f(250, 730))
        };
    }

    void flushEvents(sf::RenderWindow& window) {
        sf::Event temp;
        while (window.pollEvent(temp)) {}
    }

    void Show(sf::RenderWindow& window) {
        while (window.isOpen()) {
            sf::Event event;
            while (window.pollEvent(event)) {
                if (event.type == sf::Event::Closed)
                    window.close();
                if (event.type == sf::Event::MouseButtonReleased) {
                    for (size_t i = 0; i < buttons.size(); i++) {
                        if (buttons[i].isMouseOver(window)) {
                            if (i == 0) {
                                SnakeGame snakeGame;
                                snakeGame.run();
                            }
                            if (i == 1) {
                                Game game; 
                                game.run();
                            }
                            if (i == 2)
                                R_P_S(font).StartGame(window);
                            if (i == 3)
                                UltimateTicTacToe(font).StartGame(window);
                            if (i == 4)
                                TickTakToe(font).StartGame(window);
                            if (i == 5)
                                GuessingGame(font).StartGame(window);
                            if (i == 6)
                                window.close();

                            sf::sleep(sf::milliseconds(300));
                            flushEvents(window);
                        }
                    }
                }
            }

          
            for (auto& btn : buttons)
                btn.shape.setFillColor(btn.isMouseOver(window) ? hoverColor : secondaryColor);
 
            window.clear(backgroundColor);
            window.draw(backgroundGradient);
            window.draw(titleBox);
            sf::Text title("Game Collection", font, 52);
            title.setPosition(220, 35);
            title.setFillColor(accentColor);
            title.setStyle(sf::Text::Bold);
            window.draw(title);
            for (auto& btn : buttons)
                btn.draw(window);
            window.display();
        }
    }
};

int main() {
    RenderWindow window(VideoMode(800, 800), "Ultimate Game Collection");
    Font font;
    if (!font.loadFromFile("C:/Windows/Fonts/arial.ttf")) {
        cerr << "Error loading font!" << endl;
        return 1;
    }
    GameMenu menu(font);
    menu.Show(window);
    return 0;
}
