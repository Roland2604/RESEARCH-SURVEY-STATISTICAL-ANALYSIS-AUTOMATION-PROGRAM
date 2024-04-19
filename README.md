Source Code
#Installing of Required Systems
pip install pandas numpy matplotlib seaborn --upgrade scipy
pip install dash dash-core-components dash-html-components dash-table plotly
pip install dash dash-renderer dash-html-components dash-core-components plotly
pip install dash dash-bootstrap-components dash-table
pip install dash plotly
pip install dash dash-bootstrap-components pandas
pip install tabulate

### import libraries
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats
from scipy.stats import ttest_ind
import dash
from dash import dcc, html, dash_table
from dash.dependencies import Input, Output
import plotly.express as px
from tabulate import tabulate 
import plotly.graph_objs as go
from dash import Dash, dcc, html
from IPython.display import display, HTML, Markdown
import warnings

# This function loads survey data from an Excel file.
# It takes a filename as input and returns a DataFrame.
def load_survey_data(filename):
    try:
        # Try to read the Excel file into a DataFrame with the first row as the header.
        return pd.read_excel(filename, header=0)
    except FileNotFoundError:
        # If the file is not found, handle the exception and return None.
        return None
    except ValueError as e:
        # If a value error occurs during reading, print the error message and return None.
        print(f"Error: {e}")
        return None
    except Exception as e:
        # If any unexpected error occurs, print the error message and return None.
        print(f"Unexpected error: {e}")
        return None
    
# This function preprocesses data based on the specified choice and handles missing values.
# It takes data (a DataFrame), num_personal_info_cols (number of personal info columns),
# choice (preprocessing choice), and categorical_mappings (mapping for categorical data) as inputs.
def preprocess_data(data, num_personal_info_cols, choice, categorical_mappings):
    if num_personal_info_cols >= data.shape[1]:
        # Check if the number of personal info columns exceeds the total number of columns.
        # If so, raise a ValueError.
        raise ValueError("Number of personal info columns exceeds total columns.")
        
    if choice == 1:
        # If choice is 1, drop rows with missing values (NaN).
        data = data.dropna()
    elif choice == 2:
        # If choice is 2, fill missing values based on data type (object or numeric).
        for column in data.columns[num_personal_info_cols:]:
            if data[column].dtype == 'object':
                data[column].fillna(data[column].mode()[0], inplace=True)
            else:
                data[column].fillna(data[column].mean(), inplace=True)
    elif choice == 3:
        # If choice is 3, prompt the user for a value to fill missing data with.
        fill_value = input("Enter the value to fill missing data with: ")
        data = data.fillna(fill_value)

    # Convert non-numeric responses to their mapped values if categorical_mappings is provided.
    if categorical_mappings:
        for column in data.columns:
            if data[column].dtype == 'object':
                data[column] = data[column].map(categorical_mappings).fillna(data[column])

    return data

# Define a function to display usage guidelines
def display_usage_guidelines():
    while True:
        # Define the usage guidelines as an HTML-formatted string
        guidelines = """
        <h2 style='color:green; font-weight:bold;'>USAGE GUIDELINES:</h2> 
        <b>Kindly adhere to the following recommendations to minimize errors and achieve the utmost precision in your outcomes.</b>
        <br><br>
        1. It is recommended to run this code in Jupyter Notebook within the Anaconda environment for
        optimal accuracy, as certain features may not be available in other programming environments or languages.<br>
        2. File Format: Ensure your survey data is in Excel format (.xlsx).<br>
        3. The format of your excel datasets should be presented as follows:<br>
        """
        
        # Define sample data as a list of dictionaries
        sample_data = [
            {"Age": 15, "Gender": "Male", "Location": "Manila", "RTU is a Good School": "Agree", "The Facility of RTU is promising": "Strongly Agree", "BS Satatistics is Easy": "Disagree"},
            {"Age": 18, "Gender": "Female", "Location": "Cavite", "RTU is a Good School": "Disagree", "The Facility of RTU is promising": "Agree", "BS Satatistics is Easy": "Strongly Agree"}
        ]

        # Generate a formatted table from the sample data using the tabulate library
        table = tabulate(sample_data, headers="keys", tablefmt="html")

        # Append the table to the guidelines
        guidelines += table

        # Add an explanation about the data columns
        guidelines += """
        <p>The first columns in your Excel file should contain demographic or personal information, like Age, Gender, Location, etc.. The remaining columns should contain survey questions</p>
        <p>In the example data above, the first three (3) columns are reserved for demographic information (Age, Gender, Location). All columns following these initial three columns are dedicated to survey questions.\n</p>
        """

        # Continue with the rest of the guidelines
        guidelines += """
        <p>4. Import Libraries: Make sure to install the  required libraries at the beginning of your code:<br>
        5. Categorical Mappings: If your survey contains non-numeric responses, consider creating mappings to convert them to numeric values.<br>
        6. Interactive Dashboards: Some analysis options provide interactive dashboards. Use dropdown menus and graphs to explore data interactively.<br>
        7. Run the code and follow the prompts and instructions.<br>
        8. Enjoy Exploring Your Survey Data: This automation code simplifies survey data analysis, allowing you to gain valuable insights efficiently.<br><br>
        <b>Have you read and taken into account the provided instructions? Please respond with 'yes' or 'no'<b>:
        """
        
        # Display the guidelines in HTML format
        display(HTML(guidelines))
        
        # Prompt the user for a response
        response = input().strip().lower()
        
        # Check the user's response and return True or display error messages accordingly
        if response == 'yes':
            return True
        elif response == 'no':
            error_message = """
            <font color='red'><b>Please review and follow the provided instructions before proceeding.</b></font><br>
            <font color='red'><b>If you are having trouble following the instructions or need assistance, you can contact the creators through the contact information provided in the user manual.</b></font>
            """
            display(HTML(error_message))
        else:
            error_message = """
            <font color='red'><b>Invalid response. Please respond with ' yes' or ' no'.</b></font>
            """
            display(HTML(error_message))
            
# This function displays a menu of program capabilities and allows the user to select analyses to perform.
def display_menu():
    # Define the menu with HTML formatting.
    menu = """
    <h2 style='color:green; font-weight:bold;'>PROGRAM CAPABILITIES</h2>
    <p><b>1. DEMOGRAPHICS OF THE RESPONDENTS</b></p>
    <p>- Create an interactive table displaying demographics data for respondents, including age, gender, and location. Show measures, frequency, and percentages.</p>
    <p><b>2. CATEGORY/FACTOR ANALYSIS</b></p>
    <p>- Analyze questions that measure a single factor or category. Provide the titles and questions under each factor/category as an example.</p>
    <p><b>3. CORRELATIONAL ANALYSIS</b></p>
    <p>- Investigate the presence of positive relationships between survey questions and between factors/categories. Generate a heatmap and highlight the top 3 strong positive relationships.</p>
    <p><b>4. SIGNIFICANT DIFFERENCES BETWEEN DATAS</b></p>
    <p>- Examine if there are significant differences between respondent demographics and their responses within each factor/category. <i>This analysis should only be conducted after completing the category/factor analysis (Capability No.2).</i></p>
    <p><b>5. INDIVIDUAL ANALYSIS FOR ALL QUESTIONS</b></p>
    <p>- Perform an individual analysis of all survey questions, presenting them with interactive charts for comprehensive insights.\n</p>
    """
    
    # Display the menu
    display(HTML(menu))

    # Get user's selection
    while True:
        display(HTML("<font style='font-family:Arial;'><b>\nSelect the analysis you want to perform (e.g., '1' for Profile of the Respondents, '1,2,3' for multiple analyses, or 'all' for all analyses):</b></font> "))
        selected_analysis = input().lower()

        try:
            # Check if the input is valid (digits from 1 to 5, comma-separated, or 'all')
            if selected_analysis == 'all' or all(char.isdigit() and 1 <= int(char) <= 5 or char == ',' for char in selected_analysis.split(',')):
                # Check if '4' is selected without '2'
                selected_numbers = [int(char) for char in selected_analysis.split(',') if char.isdigit()]
                if 4 in selected_numbers and 2 not in selected_numbers:
                    raise ValueError("\n<font color='red'><b>Error:</b><b> You cannot select '4' without selecting '2'.</font></b>")
                else:
                    return selected_analysis
            else:
                raise ValueError("\n<font color='red'><b>Error:</b><b> Invalid input. Please enter a digit from 1 to 5, digits from 1 to 5 separated by a comma, or 'all'.\nExample: '1', '2,3', 'all'.</font></b>")
        except ValueError as e:
            display(Markdown(str(e)))

# This function checks for non-numeric responses in the data, provides instructions to convert them to numeric values,
# and allows the user to input the mappings.
# It takes 'data' (a DataFrame) and 'num_personal_info_cols' (number of personal info columns) as inputs.
def display_categorical_mappings(data, num_personal_info_cols):
    non_numeric_responses = set()

    # Check each column for non-numeric responses
    for column in data.columns[num_personal_info_cols:]:
        if data[column].dtype == 'object':
            non_numeric_responses.update(data[column].unique())

    # Proceed only if there are non-numeric responses
    if non_numeric_responses:
        display(HTML("<h2 style='color:green; font-weight:bold;'>NON-NUMERIC RESPONSES CONVERSIONS:</h3>"))
        non_numeric_responses_str = ', '.join(f'<u><b>{response}</b></u>' for response in non_numeric_responses)  # Format each response

        # Display the instruction, non-numeric responses, and the example conversion table
        instruction_html = f"""
        <b>\nINSTRUCTION:</b> Based on checking, your survey contains non-numeric responses: {non_numeric_responses_str}<br>
        <p>Please convert these non-numeric responses into numeric values so that the system can calculate them.</p>
        <h4 style='color:green; font-weight:bold;'>Non-numeric Response Conversion Table Example:</h3>
        """
        display(HTML(instruction_html))

        # Example data for non-numeric response conversion
        example_mapping_data = [
            ['Very Satisfied', 4],
            ['Satisfied', 3],
            ['Dissatisfied', 2],
            ['Very Dissatisfied', 1]
        ]

        # Generate an HTML table from the example data
        html_table = tabulate(example_mapping_data, headers=['Non-numeric', 'Numeric Equivalent'], tablefmt='html')
        display(HTML(html_table))

        # Ask the user to input mappings
        mapping_data = []
        for response in non_numeric_responses:
            while True:
                display(HTML(f"<p>Please enter the numeric value for <u><b>{response}</b></u>:</p>"))
                try:
                    numeric_value = int(input())
                    if numeric_value > 0:
                        mapping_data.append([response, numeric_value])
                        break  # Exit the loop if a valid input is provided
                    else:
                        raise ValueError("<font color='red'><b>Error:</b></font> Please enter a numeric value greater than 0.")
                except ValueError as e:
                    display(Markdown(f"<font color='red'><b>Error:</b> Invalid input. Please enter a valid numeric value greater than 0.</font>"))

        # Now you can create the dictionary from the mapping_data
        result_dict = {response: value for response, value in mapping_data}

        return result_dict

    # If no non-numeric responses are found, an empty dictionary is returned
    return {}


# This function analyzes personal information columns in the data and provides a profile analysis.
# It takes 'data' (a DataFrame) and 'num_personal_info_cols' (number of personal info columns) as inputs.
def analyze_personal_info(data, num_personal_info_cols):
    # Initialize an empty dictionary to store the profile analysis results.
    profile_analysis = {}
    
    # Iterate through the personal information columns.
    for column in data.columns[:num_personal_info_cols]:
        # Calculate the counts of each unique value in the column.
        counts = data[column].value_counts()
        
        # Calculate the total number of responses in the column.
        total = len(data[column])
        
        # Calculate the percentage of each unique value.
        percentages = (counts / total * 100).round(0).astype(int)
        
        # Create a DataFrame to store the analysis results.
        profile_analysis[column] = pd.DataFrame({
            'Measure': counts.index,
            'Frequency': counts.values,
            'Percentage': percentages.astype(str) + '%'  # Format percentage values as strings.
        })
    
    # Return the profile analysis results as a dictionary.
    return profile_analysis

# This function interprets a verbal response (mean) based on a reverse mapping.
# It takes 'mean' (the mean value) and 'reverse_mapping' (a mapping from mean to interpretation) as inputs.
def interpret_verbal(mean, reverse_mapping):
    # Round the mean value to the nearest integer.
    mean_rounded = round(mean)
    
    # Get the interpretation from the reverse mapping, or a default message if not found.
    return reverse_mapping.get(mean_rounded, "No interpretation available")

# This function displays a Dash DataTable for profile analysis along with a pie chart and verbal interpretation.
# It takes 'profile_analysis' (a dictionary of profile analysis data) and 'data' (the original data) as inputs.
def display_profile_table(profile_analysis, data):
    # Create a Dash web application
    app = dash.Dash(__name__)

    # Define the layout of the Dash application
    app.layout = html.Div([
        html.Div([
            dcc.Dropdown(
                id='profile-dropdown',
                options=[{'label': column, 'value': column} for column in profile_analysis.keys()],
                value=list(profile_analysis.keys())[0]
            ),
            dash_table.DataTable(
                id='profile-table',
                columns=[
                    {'name': 'Measure', 'id': 'Measure'},
                    {'name': 'Frequency', 'id': 'Frequency'},
                    {'name': 'Percentage', 'id': 'Percentage'}
                ],
                style_table={'overflowX': 'auto'},
                style_cell={'minWidth': 95},
            ),
            html.Div([
                dcc.Graph(id='profile-pie-chart', style={'height': '350px', 'width': '800px'})
            ], style={'display': 'flex', 'justify-content': 'center'}),  # Center the pie chart
            html.Div(id='verbal-interpretation', style={'color': 'black', 'text-align': 'center', 'font-weight': 'bold'})
        ]),
    ])

    # Define a callback function to update the table, verbal interpretation, and pie chart based on user's selection
    @app.callback(
        [Output('profile-table', 'data'), Output('verbal-interpretation', 'children'), Output('profile-pie-chart', 'figure')],
        [Input('profile-dropdown', 'value')]
    )
    def update_table(selected_column):
        # Retrieve the DataFrame for the selected column
        df = profile_analysis[selected_column]

        # Find the response with the highest percentage
        highest_percentage_row = df[df['Percentage'] == df['Percentage'].max()]
        highest_percentage_response = highest_percentage_row.iloc[0]['Measure']
        highest_percentage = highest_percentage_row.iloc[0]['Percentage']

        # Find the response with the lowest percentage
        lowest_percentage_row = df[df['Percentage'] == df['Percentage'].min()]
        lowest_percentage_response = lowest_percentage_row.iloc[0]['Measure']
        lowest_percentage = lowest_percentage_row.iloc[0]['Percentage']

        # Create the verbal interpretation
        verbal_interpretation = html.Div([
            html.H3(f"VERBAL INTERPRETATION [Demographic 1 : {selected_column}]", style={'color': 'green', 'text-align': 'center', 'font-weight': 'bold'}),
            html.P(f"\nAccording to the result of '{selected_column}' demographics, '{highest_percentage_response}' has the highest number of respondents with {highest_percentage} while the lowest is '{lowest_percentage_response}' with {lowest_percentage}"),
        ])

        # Create a pie chart
        fig = px.pie(df, values='Frequency', names='Measure', title=f'{selected_column} Distribution')

        return df.to_dict('records'), verbal_interpretation, fig

    # Run the Dash application on port 8050 without reloader
    app.run_server(port=8050, use_reloader=False)

# This function analyzes categories of survey questions and provides category-wise and overall analysis.
# It takes 'data' (a DataFrame), 'categories' (a dictionary of categories and their associated questions),
# 'num_personal_info_cols' (number of personal info columns), and 'reverse_categorical_mappings' (mapping for verbal responses) as inputs.
def analyze_category(data, categories, num_personal_info_cols, reverse_categorical_mappings):
    # Initialize dictionaries to store category-wise and overall analysis.
    category_analysis = {}
    overall_analysis = []
    overall_n = []
    
    # Iterate through the categories and their associated questions.
    for category, questions in categories.items():
        # Filter out invalid questions that are not present in the data columns.
        valid_questions = [q for q in questions if q in data.columns[num_personal_info_cols:]]
        
        # If no valid questions are found for the category, skip it with a warning.
        if not valid_questions:
            print(f"Warning: No valid questions found for category '{category}'. Skipping this category.")
            continue

        # Extract data for the valid questions in the category.
        category_data = data[valid_questions]
        n = len(category_data)
        
        # Calculate the mean and standard deviation for the category's questions.
        mean = category_data.mean(numeric_only=True)
        std_dev = category_data.std(numeric_only=True)

        # Create a list to store category-specific analysis data.
        analysis_data = []
        
        # For each question in the category.
        for question in valid_questions:
            # Interpret the verbal response mean value.
            interpretation = interpret_verbal(mean[question], reverse_categorical_mappings)
            analysis_data.append([question, n, f"{mean[question]:.2f}", f"{std_dev[question]:.2f}", interpretation])

        # Calculate category averages.
        total_mean = mean.mean()
        total_std_dev = std_dev.mean()
        total_interpretation = interpret_verbal(total_mean, reverse_categorical_mappings)
        analysis_data.append([f"{category} OVERALL", n, f"{total_mean:.2f}", f"{total_std_dev:.2f}", total_interpretation])
        category_analysis[category] = analysis_data

        # Collect data for overall analysis.
        overall_n.append(n)
        if analysis_data:
            overall_analysis.append([category, n, f"{total_mean:.2f}", f"{total_std_dev:.2f}", total_interpretation])

    # Add an overall row in overall analysis.
    if overall_analysis:
        overall_avg_n = np.mean(overall_n)
        overall_means = np.mean([float(row[2]) for row in overall_analysis])
        overall_stdevs = np.mean([float(row[3]) for row in overall_analysis])
        overall_interpretation = interpret_verbal(overall_means, reverse_categorical_mappings)
        overall_analysis.append(["OVERALL", overall_avg_n, f"{overall_means:.2f}", f"{overall_stdevs:.2f}", overall_interpretation])

    return category_analysis, overall_analysis

# This function displays category-wise and overall analysis using Dash.
# It takes 'category_analysis' (a dictionary of category-wise analysis data),
# 'overall_analysis' (a list of overall analysis data), and 'data' (the original data) as inputs.
def display_category_analysis(category_analysis, overall_analysis, data):
    # Create a Dash web application
    app = Dash(__name__)

    # Define the layout of the Dash application
    app.layout = html.Div([
        html.Div([
            dcc.Dropdown(
                id='category-dropdown',
                options=[{'label': category, 'value': category} for category in category_analysis.keys()],
                value=list(category_analysis.keys())[0]
            ),
            dash_table.DataTable(
                id='category-table',
                columns=[
                    {'name': 'Question', 'id': 'Question'},
                    {'name': 'N', 'id': 'N'},
                    {'name': 'Mean', 'id': 'Mean'},
                    {'name': 'Std Dev', 'id': 'Std Dev'},
                    {'name': 'Interpretation', 'id': 'Interpretation'}
                ],
                style_table={'overflowX': 'auto'},
                style_cell={'minWidth': 95},
            )
        ]),
        html.Div(
            id='verbal-interpretation',
            style={
                'margin': 'auto',
                'text-align': 'center',
            }
        )
    ])

    # Define a callback function to update the table and verbal interpretation based on the selected category
    @app.callback(
        [Output('category-table', 'data'), Output('verbal-interpretation', 'children')],
        [Input('category-dropdown', 'value')]
    )
    def update_category_table(selected_category):
        # Retrieve the analysis data for the selected category
        df = pd.DataFrame(category_analysis[selected_category], columns=['Question', 'N', 'Mean', 'Std Dev', 'Interpretation'])

        # Find the highest and lowest mean scores for questions within the category
        highest = df.iloc[df['Mean'].idxmax()]
        lowest = df.iloc[df['Mean'].idxmin()]

        # Calculate the overall mean and assessment for the category
        overall_mean = df['Mean'].astype(float).mean()
        overall_assessment = mean_to_category(overall_mean)

        # Prepare verbal interpretation content
        interpretation_content = html.Div([
            html.H3("VERBAL INTERPRETATION", style={'color': 'green', 'text-align': 'center', 'font-weight': 'bold'}),
            html.Center(html.P(f"In '{selected_category}', the highest scoring question is '{highest['Question']}' with a mean score of {highest['Mean']} ({highest['Interpretation']}), while the lowest is '{lowest['Question']}' with a score of {lowest['Mean']} ({lowest['Interpretation']}),", style={'font-weight': 'bold'})),
            html.Center(html.P(f"Overall, this category rates as {overall_assessment} with an average score of {overall_mean:.2f},", style={'font-weight': 'bold'}))
        ])

        return df.to_dict('records'), interpretation_content

    # Additional layout elements for overall analysis
    if overall_analysis:
        overall_df = pd.DataFrame(overall_analysis, columns=['Category', 'N', 'Mean', 'Std Dev', 'Interpretation'])

        # Modify the last row's category to "OVERALL"
        if not overall_df.empty:
            overall_df.at[len(overall_df) - 1, 'Category'] = "OVERALL"

        # Create an Overall Analysis Table
        overall_table = dash_table.DataTable(
            id='overall-table',
            columns=[
                {'name': 'Category', 'id': 'Category'},
                {'name': 'N', 'id': 'N'},
                {'name': 'Mean', 'id': 'Mean'},
                {'name': 'Std Dev', 'id': 'Std Dev'},
                {'name': 'Interpretation', 'id': 'Interpretation'}
            ],
            data=overall_df.to_dict('records'),
            style_table={'overflowX': 'auto'},
            style_cell={'minWidth': 95},
        )

        # Find the highest and lowest mean scores across all categories in the overall analysis
        highest = overall_df.iloc[overall_df['Mean'].astype(float).idxmax()]
        lowest = overall_df.iloc[overall_df['Mean'].astype(float).idxmin()]

        # Calculate the overall mean and assessment for the overall analysis
        overall_mean = overall_df['Mean'].astype(float).mean()
        overall_assessment = mean_to_category(overall_mean)

        # Prepare verbal interpretation content for overall analysis
        overall_interpretation_content = html.Div([
            html.H3("OVERALL VERBAL INTERPRETATION", style={'color': 'green', 'text-align': 'center', 'font-weight': 'bold'}),
            html.Center(html.P(f"In the overall analysis, the highest scoring category is '{highest['Category']}' with a mean score of {highest['Mean']} ({highest['Interpretation']}), while the lowest is '{lowest['Category']}' with a score of {lowest['Mean']} ({lowest['Interpretation']}),", style={'font-weight': 'bold'})),
            html.Center(html.P(f"Overall, the analysis rates as {overall_assessment} with an average score of {overall_mean:.2f},", style={'font-weight': 'bold'}))
        ])

        # Add Overall Analysis section to the layout
        app.layout.children.append(html.Div([html.H3('Overall Analysis'), overall_table, overall_interpretation_content]))

    # Run the Dash application on port 8051 without reloader
    app.run_server(port=8051, use_reloader=False)
    
# This function converts categorical data in a DataFrame to numeric values based on provided mappings.
# It takes 'data' (a DataFrame) and 'categories' (a dictionary of category-to-mapping pairs) as inputs.
def convert_categorical_to_numeric(data, categories):
    # Create a copy of the original data to avoid modifying the input DataFrame.
    converted_data = data.copy()
    
    # Iterate through the categories and their associated mappings.
    for category, mappings in categories.items():
        # Check if the category exists as a column in the DataFrame.
        if category in converted_data.columns:
            # Use the 'map' method to replace categorical values with their numeric equivalents.
            converted_data[category] = converted_data[category].map(mappings)
    
    # Return the DataFrame with categorical values converted to numeric values.
    return converted_data

# This function interprets a mean value using a reverse mapping.
# It rounds the mean value to the nearest integer and looks up its interpretation in the reverse mapping.
# If the rounded mean exists as a key in the reverse_mapping, it returns the corresponding interpretation; otherwise, it returns "No interpretation available."
def interpret_verbal(mean, reverse_mapping):
    mean_rounded = round(mean)  # Round the mean value to the nearest integer
    return reverse_mapping.get(mean_rounded, "No interpretation available")  # Look up the interpretation in the reverse_mapping

# This function categorizes a mean value into descriptive categories.
# It categorizes mean values based on the following thresholds:
# - If the mean_value is greater than or equal to 4, it returns 'Very High'.
# - If the mean_value is greater than or equal to 3, it returns 'High'.
# - If the mean_value is greater than or equal to 2, it returns 'Medium'.
# - Otherwise, it returns 'Low'.
def mean_to_category(mean_value):
    if mean_value >= 4:
        return 'Very High'
    elif mean_value >= 3:
        return 'High'
    elif mean_value >= 2:
        return 'Medium'
    else:
        return 'Low'
    
# This function displays combined correlation heatmaps for individual survey questions and categories,
# along with verbal interpretations for both. It visualizes the correlations between questions and
# categories in the cleaned data using heatmaps and provides insights into relationships.
def display_combined_correlation_heatmaps(cleaned_data, num_personal_info_cols, categories):
    # Set font style with Times New Roman
    font_style = {'family': 'serif', 'weight': 'bold', 'fontname': 'Times New Roman'}

    # Heatmap for Individual Questions
    survey_questions_data = cleaned_data.iloc[:, num_personal_info_cols:]
    corr_matrix_questions = survey_questions_data.corr()
    plt.figure(figsize=(12, 8))
    
    # Use a green colormap for the heatmap
    ax1 = sns.heatmap(corr_matrix_questions, annot=True, cmap='YlGnBu', fmt=".2f", cbar=False)
    ax1.set_title("Individual Question Correlation Heatmap", fontdict=font_style)
    
    # Numbering format for x-axis labels
    x_labels = [f'Q{i}' for i in range(1, len(corr_matrix_questions.columns) + 1)]
    ax1.set_xticklabels(x_labels, rotation=90, va="center", position=(0, -0.05))
    
    # Numbering format for y-axis labels
    y_labels = [f'Q{i}' for i in range(1, len(corr_matrix_questions.columns) + 1)]
    ax1.set_yticklabels(y_labels, rotation=0)
    
    ax1.xaxis.tick_top()  # Move x-axis labels to top
    plt.show()

    # Displaying verbal interpretation for Individual Questions
    display_top_relationships(corr_matrix_questions)

    # Heatmap for Categories
    category_data = pd.DataFrame()
    for category, questions in categories.items():
        category_data[category] = cleaned_data[questions].mean(axis=1)

    corr_matrix_categories = category_data.corr()
    plt.figure(figsize=(12, 8))
    
    # Use a green colormap for the heatmap
    ax2 = sns.heatmap(corr_matrix_categories, annot=True, cmap='YlGnBu', fmt=".2f", cbar=False)
    ax2.set_title("Category Correlation Heatmap", fontdict=font_style)
    
    ax2.xaxis.tick_top()  # Move x-axis labels to top
    plt.show()

    # Displaying verbal interpretation for Categories
    display_top_relationships(corr_matrix_categories)

# This function displays the top 3 strong positive relationships in a correlation matrix
# and provides verbal interpretations for each relationship.
# It takes a correlation matrix 'corr_matrix' as input and outputs the verbal interpretations in HTML format.
def display_top_relationships(corr_matrix):
    # Get all pairs of correlations from the correlation matrix
    corr_pairs = corr_matrix.unstack()

    # Sort the correlation pairs in descending order (strongest correlations first)
    sorted_pairs = corr_pairs.sort_values(kind="quicksort", ascending=False)

    # Filter out correlations with a value of 1 (self-correlations) and keep only strong positive correlations
    strong_pairs = sorted_pairs[sorted_pairs != 1]

    # Filter out repeated pairs to keep only unique top pairs
    seen_pairs = set()
    unique_top_pairs = []
    for pair, value in strong_pairs.items():
        # Check if the reverse pair (B, A) is not already seen and avoid self-correlations
        if pair[::-1] not in seen_pairs and pair[0] != pair[1]:
            seen_pairs.add(pair)
            unique_top_pairs.append((pair, value))
            if len(unique_top_pairs) == 3:
                break  # Stop after finding the top 3 unique pairs

    # Generate HTML output for the top 3 strong positive relationships
    html_output = '<h2 style="color: green; text-align: center; font-weight: bold; font-family: Times New Roman;">VERBAL INTERPRETATION - Top 3 Strong Positive Relationships</h2>'
    for pair, value in unique_top_pairs:
        # Create a verbal interpretation for each strong positive relationship
        relationship = f"The variables '{pair[0]}' and '{pair[1]}' show a strong positive correlation: {value:.2f}"
        html_output += f'<p style="text-align: center; font-weight: bold; font-family: Times New Roman;">{relationship}</p>'

    # Display the HTML output
    display(HTML(html_output))

# This function computes significant differences in means between demographic groups and category means.
# It performs t-tests to assess whether there are significant differences in category responses
# based on different demographic groups.
# The function takes 'data' (cleaned survey data), 'num_personal_info_cols' (number of demographic columns),
# and 'categories' (a dictionary of category names and their associated questions) as input.
def compute_significant_differences(data, num_personal_info_cols, categories):
    results = []
    # Calculate category means
    category_means = {category: data[questions].mean(axis=1).mean() for category, questions in categories.items()}

    # Loop through each demographic column
    for i in range(num_personal_info_cols):
        demographic_column = data.columns[i]
        unique_demographics = data[demographic_column].unique()
        
        # Loop through each category
        for category, category_mean in category_means.items():
            # Calculate mean for each demographic within the category
            for demo in unique_demographics:
                demo_data = data[data[demographic_column] == demo][categories[category]].mean(axis=1)
                demo_data = demo_data[~np.isnan(demo_data)]  # Exclude NaN values

                # Check if any column is empty
                if demo_data.empty:
                    continue

                # Perform the t-test against the overall category mean
                t_stat, p_val = stats.ttest_1samp(demo_data, category_mean)

                # Append the results
                results.append({
                    'Category': category,
                    'Demographic': demographic_column,
                    'Group': demo,
                    'N': len(demo_data),
                    'Mean': f"{demo_data.mean():.2f}",
                    'Std Dev': f"{demo_data.std(ddof=1):.4f}",
                    'T-Value': f"{t_stat:.4f}",
                    'P-Value': f"{p_val:.4f}",
                    'Significant Difference': 'Yes' if p_val < 0.05 else 'No'
                })

    return pd.DataFrame(results)


# This function displays a dashboard for significant differences analysis based on demographic groups.
# It uses Dash for creating the interactive dashboard. The dashboard includes a dropdown to select
# a demographic column, a table to display significant differences results, and a verbal interpretation.
# The function takes 'data' (cleaned survey data), 'num_personal_info_cols' (number of demographic columns),
# and 'categories' (a dictionary of category names and their associated questions) as input.
def display_significant_differences_dashboard(data, num_personal_info_cols, categories):
    # Compute significant differences using the provided function
    sig_diffs_df = compute_significant_differences(data, num_personal_info_cols, categories)

    # Initialize the Dash app
    app = dash.Dash(__name__, external_stylesheets=['https://codepen.io/chriddyp/pen/bWLwgP.css'])

    # Define the font style
    font_style = {'font-family': 'Times New Roman'}

    # Define the layout of the dashboard
    app.layout = html.Div([
        dcc.Dropdown(
            id='demographic-dropdown',
            options=[{'label': col, 'value': col} for col in data.columns[:num_personal_info_cols]],
            value=data.columns[0]
        ),
        dash_table.DataTable(
            id='sig-diffs-table',
            columns=[{"name": i, "id": i} for i in sig_diffs_df.columns],
            data=sig_diffs_df.to_dict('records'),
            style_table={'overflowX': 'auto'},
        ),
        html.Div(id='verbal-interpretation', style=font_style)
    ])

    @app.callback(
        Output('sig-diffs-table', 'data'),
        Output('verbal-interpretation', 'children'),
        [Input('demographic-dropdown', 'value')]
    )
    def update_dashboard(selected_demographic):
        # Filter the data based on the selected demographic
        filtered_data = sig_diffs_df[sig_diffs_df['Demographic'] == selected_demographic]
        
        # Filter significant differences within the selected demographic
        significant_diffs = filtered_data[filtered_data['Significant Difference'] == 'Yes']

        if significant_diffs.empty:
            verbal_summary = html.P(f"No significant differences found for the demographic: {selected_demographic}.", 
                                    style={'text-align': 'center', 'font-weight': 'bold'})
        else:
            categories_with_diffs = ', '.join(significant_diffs['Category'].unique())
            verbal_summary = html.P(f"Significant differences found for the demographic: {selected_demographic} in categories: {categories_with_diffs}.", 
                                    style={'text-align': 'center', 'font-weight': 'bold'})

        # Create the verbal interpretation section
        return filtered_data.to_dict('records'), html.Div([
            html.H3("VERBAL INTERPRETATION", style={'text-align': 'center', 'color': 'green', 'font-family': 'Times New Roman', 'font-weight': 'bold'}),
            verbal_summary
        ])

    # Run the Dash app in debug mode
    app.run_server(debug=True)
    
# This function generates a verbal interpretation for a selected survey question.
# It calculates statistics such as the highest response option, its count, percentage of total responses,
# and the mean value for the selected question. It then creates an HTML interpretation section.
# The function takes 'selected_question' (the question for which interpretation is generated)
# and 'data' (survey data containing the responses to the selected question) as input.
def generate_verbal_interpretation(selected_question, data):
    # Select the data for the chosen question
    selected_data = data[selected_question]
    
    # Calculate the option with the highest responses and its count
    highest_response_option = selected_data.value_counts().idxmax()
    highest_response_count = selected_data.value_counts().max()
    
    # Calculate the percentage of total responses for the highest response option
    total_respondents = len(selected_data)
    percentage = (highest_response_count / total_respondents) * 100
    
    # Calculate the mean value for the selected question
    mean_value = selected_data.mean()
    
    # Create a verbal interpretation section as an HTML Div
    interpretation = html.Div([
        html.H3(f"VERBAL INTERPRETATION: '{selected_question}'", style={'color': 'green', 'text-align': 'center', 'font-weight': 'bold'}),
        html.P(f"\nThe data of '{selected_question}' shows that the option with the highest responses is '{highest_response_option}' with {highest_response_count} respondents, accounting for {percentage:.2f}% of the total. The calculated mean for '{selected_question}' is {mean_value:.2f}."),
    ])

    return interpretation

def run_interactive_dashboard(data, num_personal_info_cols):
    app = dash.Dash(__name__)

    # Filter out personal info columns from survey_questions
    survey_questions = [q for q in data.columns if q not in data.columns[:num_personal_info_cols]]
    
    app.layout = html.Div([
        dcc.Dropdown(
            id='question-dropdown',
            options=[{'label': q, 'value': q} for q in survey_questions],
            value=survey_questions[0]
        ),
        dcc.Graph(id='question-graph'),
        html.Div(id='verbal-interpretation', style={'color': 'black', 'text-align': 'center', 'font-weight': 'bold'})
    ])

    @app.callback(
        [Output('question-graph', 'figure'), Output('verbal-interpretation', 'children')],
        [Input('question-dropdown', 'value')]
    )
    def update_graph(selected_question):
        filtered_data = data[selected_question].value_counts()
        fig = px.bar(
            filtered_data,
            x=filtered_data.index,
            y=filtered_data.values,
            labels={'x': selected_question, 'y': 'Counts'},
            color_discrete_sequence=['green']  # Set the color to green
        )

        # Generate the verbal interpretation based on the selected question
        interpretation = generate_verbal_interpretation(selected_question, data)

        return fig, interpretation
    
    app.run_server(port=8053, use_reloader=False)
    
# This function generates and displays an HTML heading for analysis with the specified title.
def display_analysis_heading(title):
    # Generate an HTML heading for analysis with green color and bold font weight
    analysis_heading = f"<h2 style='color: green; font-weight: bold;'>{title}</h2>"
    
    # Display the HTML heading
    display(HTML(analysis_heading))

# This function displays the title of the program with a specified console width.
def display_title(title):
    # Determine the console width for centering the title
    console_width = 140
    
    # Print the title centered within the console width
    print(title.center(console_width), "\n")
    
# Use a context manager to temporarily filter out the specific warning
with warnings.catch_warnings():
    warnings.filterwarnings("ignore", category=RuntimeWarning, message="Precision loss occurred in moment calculation due to catastrophic cancellation.")

def custom_warning_filter(message, category, filename, lineno, file=None, line=None):
    if "Precision loss occurred in moment calculation due to catastrophic cancellation" in str(message):
        return None
    else:
        return message, category, filename, lineno, line

# Apply the custom warning filter
warnings.showwarning = custom_warning_filter
    
# This is the main function of the SURVEYMASTER PRO system, which automates survey data analysis.

def main():
    # Set the title for the web page
    title = "<h1 style='color:white; background-color:green; font-size:24px; padding:10px; text-align:center;'>SURVEYMASTER PRO: SURVEY DATA ANALYSIS AUTOMATION SYSTEM</h1>"
    display(HTML(title))

    # Display usage guidelines and confirm if the user understands them
    if not display_usage_guidelines():
        return

    # Add the H2 title for dataset collection
    dataset_collection_title = "<h2 style='font-size:20px; color:green;'>COLLECTION AND CLEANING OF SURVEY DATASET</h2>"
    display(HTML(dataset_collection_title))

    # Display instructions for the user
    instructions = """
    <p style='font-family:Arial;'><b>INSTRUCTION:</b> Please ensure you have the directory path of your survey data file.</p>
    <p style='font-family:Arial;'>Your file should be in Excel (.xlsx) format. For example: 'C:/Users/YourName/Documents/survey_data.xlsx'</p>
    <p style='font-family:Arial;'>For guidance on how to find your file's directory, visit: <a href='https://www.datanumen.com/blogs/3-effective-methods-to-get-the-path-of-the-current-excel-workbook/'>here</a>.</p>
    """
    display(HTML(instructions))

    # Loop to input and load survey data file
    while True:
        try:
            display(HTML("<p><b>Enter the directory path of your survey data file:</b></p>"))
            filename = input()
            data = load_survey_data(filename)
            if data is None:
                raise ValueError("<font color='red'><b>Error:</b></font>")
            break
        except ValueError as e:
            display(Markdown(f"\n<font color='red'><b>Error:</b><b> No data loaded. Please check your file path and ensure it's an Excel file. Please try again.</font></b>"))

    # Check if data is loaded successfully
    if data is not None:
        # Input the number of columns with demographics
        while True:
            try:
                display(HTML("<font style='font-family:Arial;'><b>Enter the number of columns with demographics of the respondents, for example, age, gender, and/or location (You may refer to USAGE GUIDELINES No.2 for further instruction.):</b></font> "))
                num_personal_info_cols = int(input())
                if num_personal_info_cols < 1 or not isinstance(num_personal_info_cols, int):
                    raise ValueError("<font color='red'><b>Error:</b></font>")
                break
            except ValueError as e:
                display(Markdown(f"\n<font color='red'><b>Error:</b><b> Number of personal info/demographics columns should be greater than 0 and less than the total columns. Please enter a valid number.</font></b>"))

        # Choose how to handle missing data
        while True:
            try:
                display(HTML("<font style='font-family:Arial;'><b>Choose how to handle missing data (1: Drop, 2: Fill with mean/mode, 3: Fill with a value):</font> "))
                choice = int(input())
                if choice not in [1, 2, 3]:
                    raise ValueError("<font color='red'><b>Error:</b></font>")
                break
            except ValueError as e:
                display(Markdown(f"\n<font color='red'><b>Error:</b><b> Please enter a valid choice (1, 2, or 3).</font></b>"))
                
        # Preprocess the data based on user's choice
        cleaned_data = preprocess_data(data, num_personal_info_cols, choice, {})

        # Get user's analysis selection
        selected_analysis = display_menu()

        # Map non-numeric responses to numeric values
        categorical_mappings = display_categorical_mappings(data, num_personal_info_cols)
        cleaned_data = preprocess_data(data, num_personal_info_cols, choice, categorical_mappings)
        
        # Profile of the Respondents Analysis
        if '1' in selected_analysis or 'all' in selected_analysis:
            analysis_title = "I. PROFILE OF THE RESPONDENTS"

            # Display the analysis heading
            display_analysis_heading(analysis_title)
            
            # Display all tables for personal information or profile of the respondents
            profile_analysis = analyze_personal_info(cleaned_data, num_personal_info_cols)
            display_profile_table(profile_analysis, data)

        # Factor/Category Analysis
        if '2' in selected_analysis or 'all' in selected_analysis:
            print()  # Add a newline to separate sections
            display_analysis_heading("II. CATEGORY/FACTOR ANALYSIS")

            # Updated instruction with HTML formatting
            instruction_html = """
            <font color='black'><b>DESCRIPTION:</b></font> This program capability enables you to analyze your survey questions grouped under a single category. 
            This feature is particularly useful when you want to assess questions that measure a specific factor or category.<br><br>
            In your survey data file, each question is assigned a unique number for your convenience (Q1, Q2, Q3..). 
            This means you no longer need to copy and paste questions. Instead, you can simply input the corresponding question number."
            """

            # Display the updated instruction using HTML
            display(HTML(instruction_html))

            # Display survey questions with HTML formatting and numbering
            questions_html = ""
            for i, question in enumerate(cleaned_data.columns[num_personal_info_cols:], start=1):
                questions_html += f"Q{i}. {question}<br>"

            display(HTML(questions_html))

            # Ask for categories and questions under each category
            categories = {}
            instruction_html = "<font color='black'><b>INSTRUCTION:</b></font> Please specify the titles for each of your categories and their respective questions by entering the corresponding question numbers from the list above. Once you have listed all of the categories, simply type 'done' to continue."

            # Display the instruction with HTML formatting
            display(HTML(instruction_html))

            # Display the example input message with HTML formatting
            example_inputs_html = "<font color='black'><b> EXAMPLE:</b></font><br>     Enter the 1st category title (or 'done' to finish): ENTERTAINMENT<br>     Enter the question numbers for the ‘ENTERTAINMENT category, separated by commas. (Please refer to the numbered questions above): Q1, Q2, Q3, Q4"
            display(HTML(example_inputs_html))

            def ordinal(n):
                if 10 <= n % 100 <= 20:
                    suffix = 'th'
                else:
                    suffix = {1: 'st', 2: 'nd', 3: 'rd'}.get(n % 10, 'th')
                return f"{n}{suffix}"

            categories = {}
            category_number = 1
            BOLD = "\033[1m"
            BLACK = "\033[30m"

            # Main loop to enter categories and questions
            while True:
                try:
                    # Prompt for the category title
                    category_name = input(f"{BLACK}{BOLD}Enter the {ordinal(category_number)} category title (or 'done' to finish): {BOLD}")
                    if category_name.lower() == 'done':
                        break

                    # Prompt for question numbers
                    while True:
                        question_nums = input(f"{BLACK}{BOLD}Enter the question numbers for the '{category_name}' category, separated by commas. (Please refer to the numbered questions above. Enter as Q1, Q2, Q3, Q4): {BOLD} ")
                        try:
                            # Process question numbers and get corresponding questions
                            question_indices = []
                            for num in question_nums.split(','):
                                num = num.strip().upper()  # Convert to uppercase and remove spaces
                                if not num.startswith("Q") or not num[1:].isdigit():
                                    raise ValueError("<font color='red'><b>Error: Invalid input format. Please enter question numbers as Q1, Q2, Q3, etc. You may refer to the list provided above.</b></font>")
                                index = int(num[1:]) - 1
                                if index < 0 or index >= len(cleaned_data.columns) - num_personal_info_cols:
                                    raise ValueError(f"<font color='red'><b>Error: Invalid question number '{num}'. Please enter a valid question number within the provided list.</b></font>")
                                question_indices.append(index)

                            questions = [cleaned_data.columns[num_personal_info_cols + i] for i in question_indices]

                            # Store the category and its questions
                            categories[category_name] = questions

                            # Increment category number
                            category_number += 1
                            break  # Exit the loop if input is valid

                        except ValueError as e:
                            display(HTML(f"<font color='red'><b>{e}</b></font>"))

                        except KeyboardInterrupt:
                            print("\nProcess interrupted.")

                except KeyboardInterrupt:
                    print("\nProcess interrupted.")

            # Define reverse_categorical_mappings here if necessary
            reverse_categorical_mappings = {
                1: 'Low',
                2: 'Medium',
                3: 'High',
                4: 'Very High'
            }

            if categories:
                category_analysis, overall_analysis = analyze_category(cleaned_data, categories, num_personal_info_cols,
                                                                        reverse_categorical_mappings)
                display_category_analysis(category_analysis, overall_analysis, data)

        # Correlational Analysis
        if '3' in selected_analysis or 'all' in selected_analysis:
            display_analysis_heading("III. CORRELATIONAL ANALYSIS")

            # Display the correlation heatmap
            display_combined_correlation_heatmaps(cleaned_data, num_personal_info_cols, categories)

        # Significant Difference Analysis
        if '4' in selected_analysis or 'all' in selected_analysis:
            display_analysis_heading("IV. SIGNIFICANT DIFFERENCES BETWEEN PROFILE OF THE RESPONDENTS AND SURVEY RESPONSES")
            display_significant_differences_dashboard(cleaned_data, num_personal_info_cols, categories)

        # Individual Analysis
        if '5' in selected_analysis or 'all' in selected_analysis:
            display_analysis_heading("V. INDIVIDUAL ANALYSIS FOR ALL QUESTIONS")

            # Run interactive dashboard
            run_interactive_dashboard(data, num_personal_info_cols)
            
# Check if the script is being run as the main program
if __name__ == "__main__":
    # If so, execute the main() function
    main()



