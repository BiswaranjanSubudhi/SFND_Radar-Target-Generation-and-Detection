# SFND_Radar-Target-Generation-and-Detection
Radar Target Generation and Detection


## Implementation Steps of the 2D CFAR Process

- Slide Window through the complete Range Doppler Map
- Select the number of Training Cells in both the dimensions.
```
Tr = 12;
Td = 10;
```

-Select the number of Guard Cells in both dimensions around the Cell under test (CUT) for accurate estimation
```
Gr = 4;
Gd = 4;
```


- offset the threshold by SNR value in dB
```
offset = 1.4;
```

- a loop such that it slides the cell under test across the range doppler map by giving margins at the edges for Training and Guard Cells.
- For every iteration sum the signal level within all the training cells. To sum convert the value from logarithmic to linear using db2pow function. 
- Average the summed values for all of the training cells used. After averaging convert it back to logarithimic using pow2db.
- Further add the offset to it to determine the threshold. 
- Next, compare the signal under CUT with this threshold. If the CUT level > threshold assign it a value of 1, else equate it to 0.
- Use RDM[x,y] as the matrix from the output of 2D FFT for implementing
 
```
   RDM = RDM/max(max(RDM));

    for i = Tr+Gr+1:(Nr/2)-(Gr+Tr)
        for j = Td+Gd+1:Nd-(Gd+Td)
            noise_level = zeros(1,1);
            for p = i-(Tr+Gr) : i+(Tr+Gr)
                for q = j-(Td+Gd) : j+(Td+Gd)
                    if (abs(i-p) > Gr || abs(j-q) > Gd)
                        noise_level = noise_level + db2pow(RDM(p,q));
                    end
                end
            end
        
            threshold = pow2db(noise_level/(2*(Td+Gd+1)*2*(Tr+Gr+1)-(Gr*Gd)-1));
            threshold = threshold + offset;
            CUT = RDM(i,j);
        
            if (CUT > threshold)
                RDM(i,j) = 1;
            else
                RDM(i,j) = 0;
            end
        end
    end
```

- The process above will generate a thresholded block, which is smaller  than the Range Doppler Map as the CUT cannot be located at the edges ofmatrix. Hence,few cells will not be thresholded. To keep the map size same set those values to 0. 
```
RDM(1 : Tr + Gr, :) = 0;
RDM(Nr/2 - (Gr + Tr) + 1 : end, :) = 0;
%columns
RDM(:, 1 : Td + Gd) = 0;
RDM(:, Nd - (Gd + Td) + 1 : end) = 0;
```


- display the CFAR output using the Surf function like we did for Range Doppler Response output.
```
figure,surf(doppler_axis,range_axis,RDM);
colorbar;
```
