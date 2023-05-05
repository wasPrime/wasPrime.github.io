---
title: Genetic Algorithm
date: 2023-05-08 23:51:51
math: true
categories:
- [system_design]
tags:
- system_design
- genetic_algorithm
---

## Code Example

A genetic algorithm for optimizing the minimum value of a function.

```Python
import random
from typing import List


# Define objective function
# 定义目标函数
def objective_function(x: int) -> int:
    return x**2


# Define genetic algorithm parameters
# 定义遗传算法的参数
population_size = 50
mutation_rate = 0.1
generations = 100


# Define population initialization function
# 定义种群的初始化函数
def initialize_population(population_size: int) -> List[List[int]]:
    population = []  # size = population_size * 4
    for i in range(population_size):
        chromosome = [random.randint(0, 1) for _ in range(4)]
        population.append(chromosome)

    return population


# Define selection function
# 定义选择函数（数量减半）
def selection(population: List[List[int]]) -> List[List[int]]:
    fitness_values = []
    for chromosome in population:
        x = sum([(chromosome[i] * 2**i)
                for i in range(4)])  # x = random.randint(0, 15)
        fitness_values.append((chromosome, objective_function(x)))
    fitness_values.sort(key=lambda x: x[1])
    selected_population = [x[0]
                           for x in fitness_values[: int(population_size / 2)]]
    return selected_population


# Define crossover function
# 定义交叉函数（恢复种群数量）
def crossover(selected_population: List[List[int]]) -> List[List[int]]:
    new_population = []

    for _ in range(population_size):
        parent1 = random.choice(selected_population)
        parent2 = random.choice(selected_population)

        child = []
        for j in range(4):
            if random.random() < 0.5:
                child.append(parent1[j])
            else:
                child.append(parent2[j])
        new_population.append(child)

    return new_population


# Define mutation function
# 定义突变函数
def mutation(new_population: List[List[int]]) -> List[List[int]]:
    for i in range(population_size):
        for j in range(4):
            if random.random() < mutation_rate:
                new_population[i][j] = 1 - new_population[i][j]

    return new_population


if __name__ == "__main__":
    # Run genetic algorithm
    # 运行遗传算法
    population = initialize_population(population_size)
    for _ in range(generations):
        selected_population = selection(population)
        new_population = crossover(selected_population)
        new_population = mutation(new_population)
        population = new_population

    # Print final result
    # 打印最终结果
    fitness_values = []
    for chromosome in population:
        x = sum([(chromosome[i] * 2**i) for i in range(4)])
        fitness_values.append(objective_function(x))
    print("fitness_values:", fitness_values)
    best_fitness_value = min(fitness_values)
    print("Best fitness value:", best_fitness_value)
```

In this example, we use a simple binary string to represent each chromosome, where each gene position can take a value of 0 or 1. Our objective function is $f(x) = x^2$, and we attempt to find the minimum value of the function. We use an initialization function to generate an initial population, a selection function to select individuals with higher fitness, a crossover function to create new individuals, and a mutation function to introduce diversity. We run 100 generations and output the best fitness value.
