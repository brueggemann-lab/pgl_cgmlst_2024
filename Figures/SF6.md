# Supplementary Figure 6
```{r}
# Load libraries
library(ggplot2)
library(dplyr)

# Using the same dataframes from Figure 2a/b

# Plot
ggplot() + theme_bw() +
  geom_histogram(data=subset(m5000_ALL_sub2), aes(x=(value*100)), binwidth=0.3, fill="#5F84A2", alpha=1, lwd=0) + 
  scale_y_continuous(expand=c(0,0), trans = "log10") +
  scale_x_continuous(expand=c(0,0), limits=c(0,100)) +
  xlab("Pairwise allelic mismatches (%)") + ylab(bquote(log[10](Frequency))) +
  theme( panel.grid = element_blank(), plot.title = element_text(face="bold", color="black"), axis.line = element_line(colour = "black"), panel.border = element_blank(),
         legend.title = element_blank(),legend.position="none",axis.text.x=element_text(color="black", size=6),axis.text.y=element_text(color="black", size=6),
         axis.title.y = element_text(face="bold", color="black", size=10),axis.title.x = element_text(face="bold", color="black", size=10)) 
 ```
