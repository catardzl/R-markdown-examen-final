# R-markdown-examen-final
Examen final metabolómica 
library(pacman)
p_load("vroom", "dplyr", "ggplot2", "ggrepel", "matrixTests") # Para análisis estadístico

Volcano_data <- vroom(file = "https://raw.githubusercontent.com/ManuelLaraMVZ/Transcript-mica/main/Datos%20completos%20miRNAs%20ejercicio.csv")
head(Volcano_data)

Volcano_data2 <- Volcano_data %>% 
  filter(Type == "Target") %>% 
  select(1, 3, 4, 5, 6, 7, 8)

Control <- Volcano_data %>% 
  filter(Type == "Selected Control") %>% 
  select(1, 3, 4, 5, 6, 7, 8)

meanT1 <- mean(Control$T1)
meanT2 <- mean(Control$T2)
meanT3 <- mean(Control$T3)
meanC1 <- mean(Control$C1)
meanC2 <- mean(Control$C2)
meanC3 <- mean(Control$C3)

DCT <- Volcano_data2 %>% 
  mutate(DCT1 = T1 - meanT1, DCT2 = T2 - meanT2, DCT3 = T3 - meanT3,
         DCC1 = C1 - meanC1, DCC2 = C2 - meanC2, DCC3 = C3 - meanC3)

DosDCT <- DCT %>% 
  mutate(A = 2^-(DCT1), B = 2^-(DCT2), C = 2^-(DCT3), 
         D = 2^-(DCC1), E = 2^-(DCC2), F = 2^-(DCC3)) %>% 
  select(1, A, B, C, D, E, F)

tvalue <- row_t_welch(DosDCT[, c("A", "B", "C")], DosDCT[, c("D", "E", "F")])

FCyPV <- DosDCT %>% 
  mutate(pvalue = tvalue$pvalue) %>% 
  mutate(FCTx = (A + B + C) / 3) %>% 
  mutate(FCCx = (D + E + F) / 3) %>% 
  mutate(FC = FCTx / FCCx) %>% 
  select(1, pvalue, FC)

p_val <- 0.05
fchange_threshold <- 2

ValoresVolcanoP <- FCyPV %>% 
  mutate(LPV = -log10(pvalue),
         LFC = log2(FC)) 

downregulated <- ValoresVolcanoP %>% 
  filter(LFC < -log2(fchange_threshold)) %>% 
  filter(pvalue < p_val)

upregulated <- ValoresVolcanoP %>% 
  filter(LFC > log2(fchange_threshold)) %>% 
  filter(pvalue < p_val)

top.down <- downregulated %>% 
  arrange(pvalue) %>% 
  head(5)

top.up <- upregulated %>% 
  arrange(pvalue) %>% 
  head(5)

Volcano <- ggplot(data = ValoresVolcanoP,
                  mapping = aes(x = LFC,
                                y = LPV)) +
  geom_point(alpha = 0.8,
             color = "black") +
  labs(title = "DISEÑO: NOMBRE DEL ESTUDIANTE") +
  theme_classic() +
  xlab("Log2(FC)") +
  ylab("-Log10(p-value)") +
  geom_hline(yintercept = -log10(p_val),
             linetype = "dashed") +
  geom_vline(xintercept = c(-log2(fchange_threshold), log2(fchange_threshold)),
             linetype = "dashed") +
  geom_point(data = upregulated,
             aes(x = LFC,
                 y = LPV),
             alpha = 1,
             size = 3,
             color = "#E64B35B2") +
  geom_point(data = downregulated,
             aes(x = LFC,
                 y = LPV),
             alpha = 1,
             size = 3,
             color = "#3C5488B2") +
  geom_label_repel(data = top.up,
                   aes(x = LFC,
                       y = LPV,
                       label = miRNA),
                   max.overlaps = 50) +
  geom_label_repel(data = top.down,
                   aes(x = LFC,
                       y = LPV,
                       label = miRNA),
                   max.overlaps = 50)

print(Volcano)
