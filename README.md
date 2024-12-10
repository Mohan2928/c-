#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>
#include <stdexcept>

// Function declarations
double cosineSimilarity(const std::vector<int>& user1, const std::vector<int>& user2);
double predictRating(const std::vector<std::vector<int>>& matrix, int targetUser, int movieIndex);
std::vector<std::pair<int, double>> recommendMovies(
    const std::vector<std::vector<int>>& matrix, int targetUser, int topN);

// Cosine similarity function
double cosineSimilarity(const std::vector<int>& user1, const std::vector<int>& user2) {
    double dotProduct = 0.0, normA = 0.0, normB = 0.0;

    for (size_t i = 0; i < user1.size(); ++i) {
        dotProduct += user1[i] * user2[i];
        normA += user1[i] * user1[i];
        normB += user2[i] * user2[i];
    }

    if (normA == 0 || normB == 0) return 0; // Avoid division by zero
    return dotProduct / (sqrt(normA) * sqrt(normB));
}

// Predict rating function
double predictRating(const std::vector<std::vector<int>>& matrix, int targetUser, int movieIndex) {
    double weightedSum = 0.0, similaritySum = 0.0;

    for (size_t otherUser = 0; otherUser < matrix.size(); ++otherUser) {
        if (otherUser == targetUser || matrix[otherUser][movieIndex] == 0) continue;

        double similarity = cosineSimilarity(matrix[targetUser], matrix[otherUser]);
        weightedSum += similarity * matrix[otherUser][movieIndex];
        similaritySum += std::abs(similarity);
    }

    if (similaritySum == 0) return 0; // No similar users
    return weightedSum / similaritySum;
}

// Recommend movies function
std::vector<std::pair<int, double>> recommendMovies(
    const std::vector<std::vector<int>>& matrix, int targetUser, int topN) {
    std::vector<std::pair<int, double>> recommendations;

    for (size_t movie = 0; movie < matrix[targetUser].size(); ++movie) {
        if (matrix[targetUser][movie] == 0) { // Only predict for unrated movies
            double predictedRating = predictRating(matrix, targetUser, movie);
            recommendations.emplace_back(movie, predictedRating);
        }
    }

    // Sort recommendations by predicted rating in descending order
    std::sort(recommendations.begin(), recommendations.end(),
              [](const auto& a, const auto& b) { return a.second > b.second; });

    // Keep only the top N recommendations
    if (recommendations.size() > topN) {
        recommendations.resize(topN);
    }
    return recommendations;
}

int main() {
    try {
        // Hardcoded ratings matrix
        std::vector<std::vector<int>> ratings = {
            {1,4,0,5,0},
            {0,3,0,4,5},
            {5,0,4,0,2},
            {0,2,5,0,3},
            {3,0,0,4,0}
        };

        // Display the ratings matrix
        std::cout << "Ratings Matrix:" << std::endl;
        for (const auto& row : ratings) {
            for (int val : row) {
                std::cout << val << " ";
            }
            std::cout << std::endl;
        }

        // Input: User index and number of recommendations
        int targetUser = 0; // Recommend for the first user
        int topN = 3;       // Number of recommendations

        // Get recommendations
        auto recommendations = recommendMovies(ratings, targetUser, topN);

        // Display recommendations
        std::cout << "\nTop " << topN << " Recommendations for User" << targetUser + 1 << ":\n";
        for (const auto& rec : recommendations) {
            std::cout << "Movie" << rec.first + 1 << ": Predicted Rating = " << rec.second << std::endl;
        }
    }
    catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
