# Alzheimer-disease-detection-using-MRI

classdef AlzheimerDetectionApp2 < matlab.apps.AppBase

    
    % Properties correspond to app components
    properties (Access = public)
        UIFigure            matlab.ui.Figure
        UploadImageButton   matlab.ui.control.Button
        DetectButton        matlab.ui.control.Button
        UIAxes              matlab.ui.control.UIAxes
        ResultLabel         matlab.ui.control.Label
        SelectedImage       string
    end

    methods (Access = private)

        % --- Upload MRI Image Button ---
        function UploadImageButtonPushed(app, event)
            [file, path] = uigetfile({'*.jpg;*.jpeg;*.png;*.tif', 'Image Files'});
            if isequal(file, 0)
                uialert(app.UIFigure, 'No image selected.', 'File Error');
                return;
            end
            imgPath = fullfile(path, file);
            app.SelectedImage = imgPath;
            img = imread(imgPath);
            imshow(img, 'Parent', app.UIAxes);
            title(app.UIAxes, 'Uploaded MRI Image');
        end

        % --- Detect Alzheimer's Stage Button ---
        function DetectButtonPushed(app, event)
            if isempty(app.SelectedImage)
                uialert(app.UIFigure, 'Please upload an MRI image first.', 'Error');
                return;
            end

            % Read and preprocess image
            img = imread(app.SelectedImage);
            img = imresize(img, [224 224]);
            gray = rgb2gray(img);

            % Extract texture features (GLCM-based)
            glcm = graycomatrix(gray, 'Offset', [0 1]);
            stats = graycoprops(glcm, {'Contrast', 'Correlation', 'Energy', 'Homogeneity'});
            featureVector = [stats.Contrast, stats.Correlation, stats.Energy, stats.Homogeneity];

            % Simple rule-based logic (for demo purposes)
            score = featureVector(1)*2 + featureVector(3)*5 - featureVector(4);

            if score < 0.5
                result = 'Non-Demented';
            elseif score < 1.2
                result = 'Very Mild Demented';
            elseif score < 2.0
                result = 'Mild Demented';
            else
                result = 'Moderate Demented';
            end

            % Display result
            app.ResultLabel.Text = ['🧠 Detected Stage: ' result];
            app.ResultLabel.FontSize = 16;
            app.ResultLabel.FontColor = [0 0.4 0];
            title(app.UIAxes, result, 'Color', 'r', 'FontSize', 14);
        end
    end

    % Component initialization
    methods (Access = private)

        % Create UI components
        function createComponents(app)
            % Create UIFigure
            app.UIFigure = uifigure('Name', 'Alzheimer Detection ');
            app.UIFigure.Position = [100 100 600 400];
            app.UIFigure.Color = [0.95 0.95 0.95];

            % Create UploadImageButton
            app.UploadImageButton = uibutton(app.UIFigure, 'push');
            app.UploadImageButton.Position = [70 40 180 35];
            app.UploadImageButton.Text = '📤 Upload MRI Image';
            app.UploadImageButton.FontWeight = 'bold';
            app.UploadImageButton.ButtonPushedFcn = createCallbackFcn(app, @UploadImageButtonPushed, true);

            % Create DetectButton
            app.DetectButton = uibutton(app.UIFigure, 'push');
            app.DetectButton.Position = [350 40 180 35];
            app.DetectButton.Text = '🔍 Detect Alzheimer Stage';
            app.DetectButton.FontWeight = 'bold';
            app.DetectButton.ButtonPushedFcn = createCallbackFcn(app, @DetectButtonPushed, true);

            % Create UIAxes
            app.UIAxes = uiaxes(app.UIFigure);
            app.UIAxes.Position = [50 100 260 260];
            title(app.UIAxes, 'No Image Uploaded');
            axis(app.UIAxes, 'off');

            % Create ResultLabel
            app.ResultLabel = uilabel(app.UIFigure);
            app.ResultLabel.Position = [330 200 250 100];
            app.ResultLabel.Text = 'Result will appear here';
            app.ResultLabel.FontSize = 14;
            app.ResultLabel.FontWeight = 'bold';
            app.ResultLabel.HorizontalAlignment = 'center';
        end
    end

    % App start and construction
    methods (Access = public)
        function app = AlzheimerDetectionApp2
            createComponents(app)
        end
    end
end
