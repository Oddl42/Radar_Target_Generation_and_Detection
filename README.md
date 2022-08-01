# Radar_Target_Generation_and_Detection
Radar Target Generation and Detection using Matlab


* For steting the Initial Speed and Postion (as well as training and guard cells later) i am using the input function.

        %initRange = 100; % must be smaller than 200m
        prompt = "Please choose initial Position of target : ";
        initRange = input (prompt);
        if (initRange > maxRange )
            initRange = 200;
        elseif (initRange < 0)
            initRange = 0;
        end

        %initSpeed = 50; % in range of +/- 70m/s
        prompt = "Please choos initial Speed of target: ";
        initSpeed = input(prompt);
        if (initSpeed > maxVelocity )
            initSpeed = 70;
        elseif (initSpeed < -maxVelocity)
            initSpeed = -70;
        end

* The range for calculation of const. Velocitiy are
r(t) = r_0 + v*t

 For the Transmitted and Recvied as well as the Beat signal, i am using the formula from the sildes.
 
        for i=1:length(t)         

           % *%TODO* :
            %For each time stamp update the Range of the Target for constant velocity. 
            r_t(i) = initRange + (initSpeed * t(i));
            td(i) = 2*r_t(i) / speed_of_light;

            % *%TODO* :
            %For each time sample we need update the transmitted and
            %received signal. 
            Tx(i) = cos( 2*pi*(fc*t(i) + (slope*t(i)^2)/2) );
            Rx(i) = cos( 2*pi*(fc*(t(i)-td(i))  + (slope*(t(i)-td(i))^2) /2) );

            % *%TODO* :
            %Now by mixing the Transmit and Receive generate the beat signal
            %This is done by element wise matrix multiplication of Transmit and
            %Receiver Signal
            Mix(i) = Tx(i) .* Rx(i);

        end
        
        
The 2D CFAR i implemented the follwowing way:
        % *%TODO* :
        %Select the number of Training Cells in both the dimensions.
        prompt = "Select the number of Training Cells Tr in Range dimensions : ";
        Tr = input (prompt);

        prompt = "Select the number of Training Cells Td in Doppler dimensions : ";
        Td = input (prompt);

        % *%TODO* :
        %Select the number of Guard Cells in both dimensions around the Cell under 
        %test (CUT) for accurate estimation
        prompt = "Select the number of Guard Cells Gr in Range dimensions : ";
        Gr = input (prompt);

        prompt = "Select the number of Guard Cells Gd in Doppler dimensions : ";
        Gd = input (prompt);

        % *%TODO* :
        % offset the threshold by SNR value in dB
        prompt = "Select the offset the threshold by SNR value in dB : ";
        offset = input (prompt);

        % *%TODO* :
        %Create a vector to store noise_level for each iteration on training cells
        RDM_CARF = RDM;

        % *%TODO* :
        %design a loop such that it slides the CUT across range doppler map by
        %giving margins at the edges for Training and Guard Cells.
        %For every iteration sum the signal level within all the training
        %cells. To sum convert the value from logarithmic to linear using db2pow
        %function. Average the summed values for all of the training
        %cells used. After averaging convert it back to logarithimic using pow2db.
        %Further add the offset to it to determine the threshold. Next, compare the
        %signal under CUT with this threshold. If the CUT level > threshold assign
        %it a value of 1, else equate it to 0.
        for i = (Tr+Gr + 1) : n - (Tr+Gr)
            for j = (Td + Gd +1) : Nd - (Td + Gd)
                noise_level = zeros(1,1);
            % Use RDM[x,y] as the matrix from the output of 2D FFT for implementing
            % CFAR
                for ii = (i -Tr+Gr) : (i+Tr+Gr)
                    for jj = (j - Td+Gd) : (j+Td+Gd)
                        if (abs(i-ii)>Gr || abs(j-jj)>Gd)
                            noise_level = noise_level + db2pow(RDM(ii,jj));
                        end
                    end
                end
                %avg = 2*(Td+Gd+1)*2*(Tr+Gr+1) -(Gr*Gd) - 1;
                avg = (2*Td+2*Gd+1)*(2*Tr+2*Gr+1) - (2*Gr+1)*(2*Gd+1);
                thershold = pow2db(noise_level / avg) + offset;

                if (RDM(i,j) > thershold)
                    RDM_CARF(i,j) = 1;
                else
                    RDM_CARF(i,j) = 0;
                end
            end
        end


with the following settings for
Tr = 8
Td = 4
Gr = 4
Gd = 2
Offset for SNR = 10

i get very good Outputs for the CFAR.

% *%TODO* :
% The process above will generate a thresholded block, which is smaller 
%than the Range Doppler Map as the CUT cannot be located at the edges of
%matrix. Hence,few cells will not be thresholded. To keep the map size same
% set those values to 0. 
RDM_CARF(RDM_CARF ~=0 & RDM_CARF ~=1) = 0; 
