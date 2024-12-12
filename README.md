#include <stdio.h> 
#include <stdlib.h> 
#include <stdbool.h> 
#include <math.h> 
#include <limits.h> 
#include <time.h> 
#define MAX_DESTINATIONS 100 
// Destination structure 
typedef struct { 
 char name[50]; 
 double latitude; 
 double longitude; 
} Destination; 
// Graph structure to store distances between destinations 
typedef struct { 
 Destination *destinations[MAX_DESTINATIONS]; 
 double adj_matrix[MAX_DESTINATIONS][MAX_DESTINATIONS]; 
 int num_destinations; 
} Graph; 
// Tour structure to store the sequence of destinations 
typedef struct { 
 int *sequence; 
 int num_destinations; 
} Tour; 
// Function to create a new destination 
Destination* create_destination(const char *name, double latitude, double longitude) { 
 Destination *dest = (Destination*)malloc(sizeof(Destination)); 
 snprintf(dest->name, 50, "%s", name); 
 dest->latitude = latitude; 
 dest->longitude = longitude; 
 return dest; 
} 
// Function to calculate distance between two destinations 
double calculate_distance(Destination *dest1, Destination *dest2) { 
 double dlat = dest2->latitude - dest1->latitude; 
 double dlon = dest2->longitude - dest1->longitude; 
 return sqrt(dlat * dlat + dlon * dlon); 
} 
// Function to initialize a graph 
void initialize_graph(Graph *graph) { 
 graph->num_destinations = 0; 
 for (int i = 0; i < MAX_DESTINATIONS; i++) { 
 for (int j = 0; j < MAX_DESTINATIONS; j++) { 
 graph->adj_matrix[i][j] = (i == j) ? 0 : INFINITY; 
 } 
 } 
} 
// Function to add a destination to the graph 
void add_destination(Graph *graph, const char *name, double latitude, double longitude) 
{ 
 if (graph->num_destinations < MAX_DESTINATIONS) { 
 Destination *dest = create_destination(name, latitude, longitude); 
 graph->destinations[graph->num_destinations] = dest; 
 for (int i = 0; i < graph->num_destinations; i++) { 
 double distance = calculate_distance(dest, graph->destinations[i]); 
 graph->adj_matrix[graph->num_destinations][i] = distance; 
 graph->adj_matrix[i][graph->num_destinations] = distance; 
 } 
 
 graph->num_destinations++; 
 } else { 
 printf("Graph is full. Cannot add more destinations.\n"); 
 } 
} 
// Function to calculate total tour distance 
double total_tour_distance(Graph *graph, Tour *tour) { 
 if (tour->num_destinations < 2) return 0.0; 
 double total_distance = 0.0; 
 for (int i = 0; i < tour->num_destinations - 1; i++) { 
 total_distance += graph->adj_matrix[tour->sequence[i]][tour->sequence[i + 1]]; 
 } 
 total_distance += graph->adj_matrix[tour->sequence[tour->num_destinations - 
1]][tour->sequence[0]]; 
 return total_distance; 
} 
// Function to initialize a tour 
void initialize_tour(Tour *tour, int capacity) { 
 tour->sequence = (int*)malloc(capacity * sizeof(int)); 
 tour->num_destinations = 0; 
} 
// Function to print the tour 
void print_tour(Graph *graph, Tour *tour) { 
 printf("Tour:\n"); 
 for (int i = 0; i < tour->num_destinations; i++) { 
 Destination *dest = graph->destinations[tour->sequence[i]]; 
 printf("%d. %s (%.2f, %.2f)\n", i + 1, dest->name, dest->latitude, dest->longitude); 
 } 
 printf("Total distance: %.2f\n", total_tour_distance(graph, tour)); 
} 
// Nearest Neighbor Algorithm to construct initial tour 
void nearest_neighbor(Graph *graph, Tour *tour) { 
 if (graph->num_destinations == 0) return; 
 bool *visited = (bool*)calloc(graph->num_destinations, sizeof(bool)); 
 int current_index = 0; 
 visited[current_index] = true; 
 tour->sequence[tour->num_destinations++] = current_index; 
 while (tour->num_destinations < graph->num_destinations) { 
 double min_distance = INFINITY; 
 int nearest_index = -1; 
 for (int i = 0; i < graph->num_destinations; i++) { 
 if (!visited[i]) { 
 double distance = graph->adj_matrix[current_index][i]; 
 if (distance < min_distance) { 
 min_distance = distance; 
 nearest_index = i; 
 } 
 } 
 } 
 if (nearest_index != -1) { 
 tour->sequence[tour->num_destinations++] = nearest_index; 
 visited[nearest_index] = true; 
 current_index = nearest_index; 
 } 
 } 
 free(visited); 
} 
// 2-opt Algorithm for tour optimization 
void two_opt(Graph *graph, Tour *tour) { 
 bool improved = true; 
 while (improved) { 
 improved = false; 
 for (int i = 1; i < tour->num_destinations - 1; i++) { 
 for (int j = i + 1; j < tour->num_destinations; j++) { 
 double delta = graph->adj_matrix[tour->sequence[i - 1]][tour->sequence[j]] + 
 graph->adj_matrix[tour->sequence[i]][tour->sequence[(j + 1) % tour-
>num_destinations]] - 
 graph->adj_matrix[tour->sequence[i - 1]][tour->sequence[i]] - 
 graph->adj_matrix[tour->sequence[j]][tour->sequence[(j + 1) % tour-
>num_destinations]]; 
 if (delta < 0) { 
 // Reverse the tour segment between i and j 
 while (i < j) { 
 int temp = tour->sequence[i]; 
 tour->sequence[i] = tour->sequence[j]; 
 tour->sequence[j] = temp; 
 i++; 
 j--; 
 } 
 improved = true; 
 } 
 } 
 } 
 } 
} 
int main() { 
 Graph graph; 
 initialize_graph(&graph); 
 Tour tour; 
 initialize_tour(&tour, MAX_DESTINATIONS); 
 int num_destinations; 
 char choice; 
 printf("Do you want to create random destinations (Y/N)? "); 
 scanf(" %c", &choice); 
 if (choice == 'Y' || choice == 'y') { 
 printf("Enter the number of random destinations: "); 
 scanf("%d", &num_destinations); 
 // Create random destinations 
 srand(time(NULL)); 
 for (int i = 0; i < num_destinations; i++) { 
 char name[50]; 
 sprintf(name, "Destination %d", i + 1); 
 double latitude = (double)rand() / RAND_MAX * 180.0 - 90.0; 
 double longitude = (double)rand() / RAND_MAX * 360.0 - 180.0; 
 add_destination(&graph, name, latitude, longitude); 
 } 
 } else { 
 printf("Enter the number of custom destinations: "); 
 scanf("%d", &num_destinations); 
 // Input custom destinations 
 for (int i = 0; i < num_destinations; i++) { 
 char name[50]; 
 double latitude, longitude; 
 printf("Enter name of destination %d: ", i + 1); 
 scanf("%s", name); 
 printf("Enter latitude of destination %d: ", i + 1); 
 scanf("%lf", &latitude); 
 printf("Enter longitude of destination %d: ", i + 1); 
 scanf("%lf", &longitude); 
 add_destination(&graph, name, latitude, longitude); 
 } 
 } 
 if (graph.num_destinations > 1) { 
 // Construct initial tour using Nearest Neighbor Algorithm 
 nearest_neighbor(&graph, &tour); 
 // Print initial tour 
 printf("\nInitial Tour:\n"); 
 print_tour(&graph, &tour); 
 // Optimize tour using 2-opt Algorithm 
 two_opt(&graph, &tour); 
 // Print optimized tour 
 printf("\nOptimized Tour:\n"); 
 print_tour(&graph, &tour); 
 } else { 
 printf("Not enough destinations to create a tour.\n"); 
 } 
 // Free allocated memory 
 for (int i = 0; i < graph.num_destinations; i++) { 
 free(graph.destinations[i]); 
 } 
 free(tour.sequence); 
 return 0; 
} 
