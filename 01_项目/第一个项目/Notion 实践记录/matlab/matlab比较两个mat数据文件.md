---
title: "matlab比较两个mat数据文件"
publish: false
tags: ["待整理"]
---
# matlab比较两个mat数据文件

```matlab
%compare two .mat files
%change the two path and design your table should work out 
%also dont forget to change the name of variable you want to calculate

%define the absolute path
folder1 = 'D:\M_PJ\output\20240718\processedmat';
folder2 = 'C:\Users\huang\Documents\20240724\mazda_cochlear_model\outputs\processedmat';
output_csv = 'results.csv';

files1 = dir(fullfile(folder1, '*.mat'));
files2 = dir(fullfile(folder2, '*.mat'));

fileNames1 = {files1.name};
fileNames2 = {files2.name};

commonFileNames = intersect(fileNames1, fileNames2);%define the file to compare

% define and initialize the results table
results = table('Size', [0 13], ...
    'VariableTypes', {'string', 'double', 'double', 'double', 'double', 'double', 'double', 'double', 'double', 'double', 'double', 'double', 'double'}, ...
    'VariableNames', {'FileName', 'LeftMaxAbsDiff', 'LeftMaxValueRange', 'LeftErrorRate', 'LeftCorrelation', ...
    'RightMaxAbsDiff', 'RightMaxValueRange', 'RightErrorRate', 'RightCorrelation', ...
    'FCMaxAbsDiff', 'FCMaxValueRange', 'FCErrorRate', 'FCCorrelation'});

for i = 1:length(commonFileNames)
    
    data1 = load(fullfile(folder1, commonFileNames{i}));
    data2 = load(fullfile(folder2, commonFileNames{i}));
    
    vars1 = fieldnames(data1);
    vars2 = fieldnames(data2);
    
    if ~isequal(vars1, vars2)
        fprintf('Not all parameters are the same for %s\n', commonFileNames{i});
        continue;
    end
    
    for j = 1:length(vars1)
        var1 = data1.(vars1{j});
        var2 = data2.(vars1{j});
        
        if isstruct(var1) && isstruct(var2) && all(isfield(var1, {'left', 'right', 'fc'})) && all(isfield(var2, {'left', 'right', 'fc'}))
            % left_diff
            max_abs_diff_left = max(abs(var1.left(:) - var2.left(:)));
            max_value_range_left = max(var1.left(:)) - min(var1.left(:));
            left_error_rate = max_abs_diff_left / max_value_range_left;
            left_correlation = corr(var1.left(:), var2.left(:));

            % right_diff
            max_abs_diff_right = max(abs(var1.right(:) - var2.right(:)));
            max_value_range_right = max(var1.right(:)) - min(var1.right(:));
            right_error_rate = max_abs_diff_right / max_value_range_right;
            right_correlation = corr(var1.right(:), var2.right(:));

            % fc_diff
            max_abs_diff_fc = max(abs(var1.fc(:) - var2.fc(:)));
            max_value_range_fc = max(var1.fc(:)) - min(var1.fc(:));
            fc_error_rate = max_abs_diff_fc / max_value_range_fc;
            fc_correlation = corr(var1.fc(:), var2.fc(:));

            % Add to the results table
            new_row = {commonFileNames{i}, ...
                max_abs_diff_left, max_value_range_left, left_error_rate, left_correlation, ...
                max_abs_diff_right, max_value_range_right, right_error_rate, right_correlation, ...
                max_abs_diff_fc, max_value_range_fc, fc_error_rate, fc_correlation};
            results = [results; new_row];
        else
            fprintf('file1 %s and file2 %s %s are not structures with left, right, and fc fields\n', commonFileNames{i}, commonFileNames{i}, vars1{j});
        end
    end
end

writetable(results, output_csv);
fprintf('Results saved to %s\n', output_csv);

```
