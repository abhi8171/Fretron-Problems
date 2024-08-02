# Fretron-Problems
# Problem has been solved by the help of chatGPT.
#problem 1

#include <iostream>
#include <vector>
#include <tuple>

using namespace std;

// Define a type for coordinates
using Point = pair<int, int>;

// Function to check if two line segments intersect
bool doIntersect(Point p1, Point q1, Point p2, Point q2) {
    // Utility function to check if point q lies on segment 'pr' 
    auto onSegment = [](Point p, Point q, Point r) {
        return q.first <= max(p.first, r.first) &&
               q.first >= min(p.first, r.first) &&
               q.second <= max(p.second, r.second) &&
               q.second >= min(p.second, r.second);
    };

    // Calculate the orientation of the triplet (p, q, r)
    auto orientation = [](Point p, Point q, Point r) {
        int val = (q.second - p.second) * (r.first - q.first) -
                  (q.first - p.first) * (r.second - q.second);
        return (val == 0) ? 0 : (val > 0) ? 1 : 2;
    };

    int o1 = orientation(p1, q1, p2);
    int o2 = orientation(p1, q1, q2);
    int o3 = orientation(p2, q2, p1);
    int o4 = orientation(p2, q2, q1);

    // General case
    if (o1 != o2 && o3 != o4) return true;

    // Special cases
    if (o1 == 0 && onSegment(p1, p2, q1)) return true;
    if (o2 == 0 && onSegment(p1, q2, q1)) return true;
    if (o3 == 0 && onSegment(p2, p1, q2)) return true;
    if (o4 == 0 && onSegment(p2, q1, q2)) return true;

    return false; // Doesn't fall in any of the above cases
}

// Function to plot the paths
void plotPaths(const vector<vector<Point>>& flights) {
    // Example printing paths for illustration
    for (int i = 0; i < flights.size(); ++i) {
        cout << "Flight " << i + 1 << " Path: ";
        for (const auto& point : flights[i]) {
            cout << "(" << point.first << "," << point.second << ") ";
        }
        cout << endl;
    }
}

// Main function
int main() {
    // Example flight paths
    vector<vector<Point>> flights = {
        {{1, 1}, {2, 2}, {3, 3}},
        {{1, 1}, {2, 4}, {3, 2}},
        {{1, 1}, {4, 2}, {3, 4}}
    };

    // Display paths
    plotPaths(flights);

    // Check for intersections
    for (size_t i = 0; i < flights.size(); ++i) {
        for (size_t j = i + 1; j < flights.size(); ++j) {
            const auto& flight1 = flights[i];
            const auto& flight2 = flights[j];
            for (size_t k = 0; k < flight1.size() - 1; ++k) {
                for (size_t l = 0; l < flight2.size() - 1; ++l) {
                    if (doIntersect(flight1[k], flight1[k + 1], flight2[l], flight2[l + 1])) {
                        cout << "Intersection detected between Flight " << i + 1 << " and Flight " << j + 1 << endl;
                    }
                }
            }
        }
    }

    return 0;
}


#Problem 2

#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
#include <iomanip>

using namespace std;

// Function to distribute apples proportionally
void distributeApples(const vector<int>& apples, int ramShare, int shamShare, int rahimShare) {
    int totalWeight = accumulate(apples.begin(), apples.end(), 0);
    int ramWeight = (totalWeight * ramShare) / 100;
    int shamWeight = (totalWeight * shamShare) / 100;
    int rahimWeight = (totalWeight * rahimShare) / 100;

    vector<int> ramApples, shamApples, rahimApples;
    vector<int> remainingApples = apples;

    sort(remainingApples.begin(), remainingApples.end(), greater<int>());

    for (int weight : remainingApples) {
        if (ramWeight >= weight) {
            ramApples.push_back(weight);
            ramWeight -= weight;
        } else if (shamWeight >= weight) {
            shamApples.push_back(weight);
            shamWeight -= weight;
        } else if (rahimWeight >= weight) {
            rahimApples.push_back(weight);
            rahimWeight -= weight;
        }
    }

    // Print the results
    cout << "Distribution Result:" << endl;
    cout << "Ram: ";
    for (int weight : ramApples) cout << weight << " ";
    cout << endl;

    cout << "Sham: ";
    for (int weight : shamApples) cout << weight << " ";
    cout << endl;

    cout << "Rahim: ";
    for (int weight : rahimApples) cout << weight << " ";
    cout << endl;
}

int main() {
    vector<int> apples;
    int weight;

    cout << "Enter apple weight in gram (-1 to stop): ";
    while (cin >> weight && weight != -1) {
        apples.push_back(weight);
        cout << "Enter apple weight in gram (-1 to stop): ";
    }

    // Define the proportion of contribution for each person
    int ramShare = 50;
    int shamShare = 30;
    int rahimShare = 20;

    distributeApples(apples, ramShare, shamShare, rahimShare);

    return 0;
}
#Problem 3

#include <iostream>
#include <vector>
#include <unordered_set>
#include <string>
#include <sstream>

using namespace std;

// Define the directions for the castle to move
enum Direction {
    RIGHT, DOWN, LEFT, UP
};

// Structure to represent a position on the chessboard
struct Position {
    int x, y;
    
    bool operator==(const Position& other) const {
        return x == other.x && y == other.y;
    }
    
    // Custom hash function for Position
    struct HashFunction {
        size_t operator()(const Position& p) const {
            return hash<int>()(p.x) ^ hash<int>()(p.y);
        }
    };
};

// Function to convert a position to string
string positionToString(const Position& p) {
    return to_string(p.x) + "," + to_string(p.y);
}

// Check if the position is within the board limits
bool isValid(int x, int y) {
    return x >= 1 && x <= 8 && y >= 1 && y <= 8;
}

// Recursively find all unique paths for the specialized castle
void findPaths(Position start, Position current, Direction dir, const unordered_set<Position, Position::HashFunction>& soldiers, 
                unordered_set<Position, Position::HashFunction>& visited, vector<Position>& path, vector<vector<Position>>& allPaths) {
    if (current == start && visited.size() == soldiers.size()) {
        // If we return to start and have visited all soldiers, record the path
        allPaths.push_back(path);
        return;
    }
    
    // Try to move in the current direction
    int dx[] = {0, 1, 0, -1}; // Movement in x direction for RIGHT, DOWN, LEFT, UP
    int dy[] = {1, 0, -1, 0}; // Movement in y direction for RIGHT, DOWN, LEFT, UP
    
    // Check the current movement direction
    int nx = current.x + dx[dir];
    int ny = current.y + dy[dir];
    
    if (isValid(nx, ny)) {
        Position nextPos = {nx, ny};
        if (soldiers.find(nextPos) != soldiers.end()) {
            // If there is a soldier at the new position
            // Mark the soldier as visited and remove it
            visited.insert(nextPos);
            path.push_back(nextPos);
            findPaths(start, nextPos, (dir + 1) % 4, soldiers, visited, path, allPaths);
            path.pop_back();
            visited.erase(nextPos);
        }
    }
    
    // Attempt to jump over soldiers in the current direction
    nx = current.x + 2 * dx[dir];
    ny = current.y + 2 * dy[dir];
    Position jumpPos = {current.x + dx[dir], current.y + dy[dir]};
    if (isValid(nx, ny) && soldiers.find(jumpPos) == soldiers.end()) {
        Position jumpEnd = {nx, ny};
        if (soldiers.find(jumpEnd) != soldiers.end()) {
            // If there's a soldier to jump over and end position has a soldier
            visited.insert(jumpEnd);
            path.push_back(jumpEnd);
            findPaths(start, jumpEnd, (dir + 1) % 4, soldiers, visited, path, allPaths);
            path.pop_back();
            visited.erase(jumpEnd);
        }
    }
}

// Main function
int main() {
    int numSoldiers;
    cout << "Enter the number of soldiers: ";
    cin >> numSoldiers;
    
    unordered_set<Position, Position::HashFunction> soldiers;
    cout << "Enter coordinates for each soldier:" << endl;
    for (int i = 0; i < numSoldiers; ++i) {
        int x, y;
        char comma;
        cin >> x >> comma >> y;
        soldiers.insert({x, y});
    }
    
    Position castle;
    cout << "Enter the coordinates for your 'special' castle: ";
    cin >> castle.x >> comma >> castle.y;
    
    vector<vector<Position>> allPaths;
    vector<Position> path;
    unordered_set<Position, Position::HashFunction> visited;
    
    // Start searching for paths from the castle position
    findPaths(castle, castle, RIGHT, soldiers, visited, path, allPaths);
    
    // Print all unique paths
    cout << "Thanks. There are " << allPaths.size() << " unique paths for your 'special_castle'" << endl;
    
    for (int i = 0; i < allPaths.size(); ++i) {
        cout << "Path " << (i + 1) << endl;
        cout << "========" << endl;
        cout << "Start (" << castle.x << "," << castle.y << ")" << endl;
        for (const auto& pos : allPaths[i]) {
            cout << "Kill (" << pos.x << "," << pos.y << "). Turn Left" << endl;
        }
        cout << "Arrive (" << castle.x << "," << castle.y << ")" << endl;
        cout << endl;
    }
    
    return 0;
}
