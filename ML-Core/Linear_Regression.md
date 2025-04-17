```python
x = 0  # starting point
learning_rate = 0.1
trajectory = []

for _ in range(30):
    grad = 2 * (x - 5)
    x = x - learning_rate * grad
    y = (x - 5) ** 2 + 3
    trajectory.append((x, y))

# Plotting the result
X = np.arange(0, 10, 0.1)
Y = (X - 5) ** 2 + 3
plt.plot(X, Y, label='y = (x - 5)^2 + 3')
for x_t, y_t in trajectory:
    plt.scatter(x_t, y_t, color='red')
plt.title("Gradient Descent Steps")
plt.xlabel("x")
plt.ylabel("y")
plt.legend()
plt.show()
```
