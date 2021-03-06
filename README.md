```{r, eval=FALSE, include=TRUE}
"Protocolo:
 
 1. Daniel Felipe Villa Rengifo
 
 2. Lenguaje: R
 
 3. Tema: Maquinas de Vector Soporte [Parte 3]
 
 4. Fuentes:  
    https://www.datacamp.com/community/tutorials/support-vector-machines-r"
```

```{r}
# Guardamos los OUTPUTS:
sink("OUTPUTS.txt")

# Volvemos a utilizar el .RData para los datos de anteriores Replits:
#load("~/R/100DaysOfCode/Dia_98_22junio/.RData")
```


# Evaluación del modelo [parte 2]

```{r}
# Maatriz de Confusión:
library(e1071)
library(caret)
library(dplyr)

print("# Maatriz de Confusión:")
confusionMatrix(predict(modelo_svmP, datosOJ_test), datosOJ_test$Purchase)


paste("Observaciones de test mal clasificadas:", 
      100 * mean(datosOJ_test$Purchase != predict(modelo_svmP, datosOJ_test)) %>%
        round(digits = 4), "%")
```

# Kernel radial

## Ajuste del modelo

```{r}
# Cargamos una semilla
set.seed(325)

# Ajustamos el modelo
tuning <- tune(svm, Purchase ~ ., data = datosOJ_train, 
               kernel = "radial", 
               ranges = list(cost = c(0.001, 0.01, 0.1, 1, 5, 10, 15), 
                             gamma = c(0.01, 0.1, 1, 5, 10)), 
               scale = TRUE)

# Resumen del ajuste del modelo
print("# Resumen del ajuste del modelo")
summary(tuning)

#Con un kernel radial, los hiperparámetros que reducen el error de validación son coste = 5, gamma = 0,01.

png(filename = "ajustemodelo.png")

# Ahora graficamos los ajustes hechos en el modelo
ggplot(data = tuning$performances, aes(x = cost, y = error, color = factor(gamma))) +
  geom_line() +
  geom_point() +
  labs(title = "Error de validación ~ hiperparámetro C y gamma") +
  theme_bw() +
  theme(plot.title = element_text(hjust = 0.5)) +
  theme(legend.position = "bottom")

dev.off()

# Modelo SVM con kernel radial
modelo_svmR <- svm(Purchase ~ . , data = datosOJ_train, 
                   kernel = "radial", 
                   cost = 5, 
                   gamma = 0.01, 
                   scale = TRUE)


## Evaluación del modelo anterior:

# Matriz de confusion y métricas en test
print("# Matriz de confusion y métricas en test")
confusionMatrix(predict(modelo_svmR, datosOJ_test), datosOJ_test$Purchase)

paste("Observaciones de test mal clasificadas:", 
       100 * mean(datosOJ_test$Purchase != predict(modelo_svmR, datosOJ_test)) %>%
           round(digits = 4), "%")

sink()
```
