clear all; close all; clc;

%% Input images
folder_path = 'C:\Users\Zhaoliang Hou\Desktop\Correlation_analysis'; % input foler()
image_files = dir(fullfile(folder_path, '*.tif'));

% Check if images were found
if isempty(image_files)
    error('No .tif files found in specified folder.');
end

% Process up to 2 images
n = min(2, length(image_files)); 

% Preallocate variables
im_data = cell(n, 1);
X_resolution = zeros(n, 1);
Y_resolution = zeros(n, 1);

% Load and preprocess images
for i = 1:n
    % Read image
    full_path = fullfile(folder_path, image_files(i).name);
    img = imread(full_path);
    
    % Remove border pixels
    cropped_img = img(2:end-1, 2:end-1);
    
    % Store processed image
    im_data{i} = cropped_img;
    [X_resolution(i), Y_resolution(i)] = size(cropped_img);
    
    % Display original image
    figure(i);
    imshow(img);
    title(['Original Image ', num2str(i)]);
end

%% Image splitting parameters
X_blocks_no = 3;     % How many rows you want
Y_blocks_no = 6;   % How many columns you want
% Calculate block dimensions
block_Xstep = floor(X_resolution ./ X_blocks_no);
block_Ystep = floor(Y_resolution ./ Y_blocks_no);
% Verify block sizes are integer values
if any(mod(X_resolution, X_blocks_no) ~= 0) || any(mod(Y_resolution, Y_blocks_no) ~= 0)
    warning('Block sizes may not be perfectly divisible; blocks will be truncated.');
end

%% Split and save images
output_path = 'C:\Users\Zhaoliang Hou\Desktop\Correlation_analysis\Test';

for i = 1:n
    % Create unique output folder for each image
    [~, base_name] = fileparts(image_files(i).name);
    save_folder = fullfile(output_path, [base_name '_blocks']);
    if ~exist(save_folder, 'dir')
        mkdir(save_folder);
    end
    
    % Get current image and initialize block counter
    current_img = im_data{i};
    block_count = 1;
    
    % Split image into blocks
    for x_block = 0:X_blocks_no-1
        for y_block = 0:Y_blocks_no-1
            % Calculate block coordinates
            x_start = x_block * block_Xstep(i) + 1;
            x_end = min((x_block + 1) * block_Xstep(i), X_resolution(i));
            y_start = y_block * block_Ystep(i) + 1;
            y_end = min((y_block + 1) * block_Ystep(i), Y_resolution(i));
            
            % Extract block
            img_block = current_img(x_start:x_end, y_start:y_end);
            
            % Save block
            block_name = sprintf('Block_%03d.tif', block_count);
            imwrite(img_block, fullfile(save_folder, block_name));
            
            block_count = block_count + 1;
        end
    end
end

disp('Processing complete!');
