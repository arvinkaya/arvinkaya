import random
import string

# Konfigurasi algoritma genetika
TARGET_NAME = "Arvin Permana Putra"
POPULATION_SIZE = 200
MUTATION_RATE = 0.02
TOURNAMENT_SIZE = 5

# Karakteristik target untuk pencarian populasi
TARGET_HEIGHT = 173  # cm
TARGET_WEIGHT = 80   # kg
TARGET_AGE = 19      # tahun

class Individual:
    def __init__(self, chromosome):
        self.chromosome = chromosome
        self.height = random.randint(150, 200)  # cm
        self.weight = random.randint(45, 100)   # kg
        self.age = random.randint(18, 60)       # tahun
        self.name_fitness = self.calculate_name_fitness()
        self.attribute_fitness = self.calculate_attribute_fitness()
    
    def calculate_name_fitness(self):
        return sum(1 for expected, actual in zip(TARGET_NAME, self.chromosome) if expected == actual)

    def calculate_attribute_fitness(self):
        height_diff = abs(self.height - TARGET_HEIGHT)
        weight_diff = abs(self.weight - TARGET_WEIGHT)
        age_diff = abs(self.age - TARGET_AGE)
        return 1 / (1 + height_diff + weight_diff + age_diff)

    def __lt__(self, other):
        return (self.name_fitness, self.attribute_fitness) < (other.name_fitness, other.attribute_fitness)

class Population:
    def __init__(self, size):
        self.individuals = [self.create_individual() for _ in range(size)]
    
    @staticmethod
    def create_individual():
        chromosome = ''.join(random.choice(string.ascii_letters + ' ') for _ in range(len(TARGET_NAME)))
        return Individual(chromosome)

    def get_fittest(self, n):
        return sorted(self.individuals, key=lambda x: (x.name_fitness, x.attribute_fitness), reverse=True)[:n]

    def tournament_selection(self):
        tournament = random.sample(self.individuals, TOURNAMENT_SIZE)
        return max(tournament, key=lambda x: (x.name_fitness, x.attribute_fitness))

    def evolve(self):
        new_population = []
        elite_size = POPULATION_SIZE // 20
        new_population.extend(self.get_fittest(elite_size))

        while len(new_population) < POPULATION_SIZE:
            parent1 = self.tournament_selection()
            parent2 = self.tournament_selection()
            child = self.crossover(parent1, parent2)
            child = self.mutate(child)
            new_population.append(child)

        self.individuals = new_population

    @staticmethod
    def crossover(parent1, parent2):
        split = random.randint(0, len(TARGET_NAME) - 1)
        child_chromosome = parent1.chromosome[:split] + parent2.chromosome[split:]
        child = Individual(child_chromosome)
        # Crossover untuk atribut
        child.height = (parent1.height + parent2.height) // 2
        child.weight = (parent1.weight + parent2.weight) // 2
        child.age = (parent1.age + parent2.age) // 2
        return child

    @staticmethod
    def mutate(individual):
        mutated_chromosome = ''.join(
            c if random.random() > MUTATION_RATE else random.choice(string.ascii_letters + ' ')
            for c in individual.chromosome
        )
        mutated_individual = Individual(mutated_chromosome)
        # Mutasi untuk atribut
        if random.random() < MUTATION_RATE:
            mutated_individual.height += random.randint(-5, 5)
        if random.random() < MUTATION_RATE:
            mutated_individual.weight += random.randint(-3, 3)
        if random.random() < MUTATION_RATE:
            mutated_individual.age += random.randint(-2, 2)
        
        # Memastikan nilai tetap dalam batas yang masuk akal
        mutated_individual.height = max(150, min(200, mutated_individual.height))
        mutated_individual.weight = max(45, min(100, mutated_individual.weight))
        mutated_individual.age = max(18, min(60, mutated_individual.age))
        
        return mutated_individual

def genetic_algorithm():
    population = Population(POPULATION_SIZE)
    generation = 0
    best_name_fitness = 0
    best_attribute_fitness = 0

    while True:
        fittest = population.get_fittest(1)[0]
        
        if fittest.name_fitness > best_name_fitness or fittest.attribute_fitness > best_attribute_fitness:
            best_name_fitness = fittest.name_fitness
            best_attribute_fitness = fittest.attribute_fitness
            print(f"Generasi {generation}:")
            print(f"  Nama Terbaik = '{fittest.chromosome}' (Fitness: {best_name_fitness}/{len(TARGET_NAME)})")
            print(f"  Atribut Terbaik - Tinggi: {fittest.height}cm, Berat: {fittest.weight}kg, Umur: {fittest.age} tahun")
            print(f"  Attribute Fitness: {best_attribute_fitness:.4f}")

        if fittest.chromosome == TARGET_NAME and fittest.attribute_fitness > 0.9:
            print(f"\nTarget ditemukan pada generasi {generation}:")
            print(f"  Nama: '{fittest.chromosome}'")
            print(f"  Tinggi: {fittest.height}cm, Berat: {fittest.weight}kg, Umur: {fittest.age} tahun")
            break

        population.evolve()
        generation += 1

if __name__ == "__main__":
    print(f"Mencari individu dengan nama: '{TARGET_NAME}'")
    print(f"dan karakteristik: Tinggi: {TARGET_HEIGHT}cm, Berat: {TARGET_WEIGHT}kg, Umur: {TARGET_AGE} tahun")
    print("Memulai algoritma genetika...")
    genetic_algorithm()
