colebrook-hands-off
===================

#include <stdio.h>          /* The libraries needed for the code below.*/
#include <string.h>
#include <grx20.h>
#include <math.h>

const int pressure_at_sea_level = 101325;
const float gravitational_constant = 6.67384E-11;
const int radius_of_earth = 6378100;
const float mass_of_earth = 5.9726E24;
const float gas_constant = 8.31;    /*1.3806488 * pow(10, -23);*/
const double PI = 3.141592654;
float time;
float density;
float mass;
float gravity;
float altitude;
float drag;
float acceleration;
float velocity;
int column_x, column_y;
 
const int   MAXROW = 250, MAXCOL = 8, COLt = 1, COLRo = 2, COLdrag = 3, COLa = 4, COLv = 5, COLh = 6, COLg = 7, COLm = 8;	


typedef struct	RSIMStructure {     /* Defining the data structure that is used throughout the program.*/               
    int	currentrow;                     /* The array contained has a fixed number of rows and columns, where the columns are for */
    float table[250][8]; 
    float time;              /* specific variables.*/                  
 }  RSIMType;


/*int Menu();
int ValidateData(), DrawText(int x, int y, char * graphtitle, int xAlign, int yAlign), ChangeParameters(int payload, int number_of_boosters, int inert_mass, int centaur_engine_type, int i);

RSIMType ClearDataTable(RSIMType datatable), AddData(RSIMType datatable);

void DisplayDataTable(RSIMType datatable, int fromrow, int torow), plot_graph( int column_x, int column_y, RSIMType datatable, float max, float xborderless, float yborderless, int x_origin, int y_origin);
void Graph_plotter(int column_x, int column_y, RSIMType datatable,  int max, float xborderless, float yborderless, int x_origin, int y_origin);  
   */
char * displaytitle = "Atlas V 400 Rocket Simulation Data", * graph1yalabel = "Acceleration (m/s^2)",* graph1yblabel = "Velocity (m/s^1)", * graph1xlabel = "Time (s)", * graph2ylabel = "Altitude (m)", * graph2xlabel = "Time (s)";


/* Above is the declaration of the functions used within the program, along with the default axis labels within global character arrays.*/










int Menu() {                                            /* Function that displays the list of options to the user.*/
    int option1;
    printf("\n\nThis is a program to simulate a rocket launch\n\n");
    printf("1.\tAdd data.\n");
    printf("2.\tDisplay experimental and calculated data.\n");
    printf("3.\tClear data set in table format\n");
    printf("4.\tChange parameters\n");
    printf("5.\tPlot graphs\n");
    printf("6.\tEnd program\n\n");
    printf("Please select: ");
    option1 = ValidateData();          /* The validate function is called here.*/
    return option1; }                    /* The user's choice is returned to the main function.*/



int ValidateData() {                                       /* This function validates the user's input, and displays an error message*/
    int selection;                                          /* if invalid.*/
    while((scanf("%d", &selection) != 1)) {                 /* The function checks for a numerical input - scanf will return a '1' if so.*/
      fflush(stdin);                                        /* The buffer is flushed so that the return key isn't registered as an input.*/
      printf("\nInvalid entry, please try again.\n\nPlease select: "); }
    return selection; }

void  DisplayDataTable(RSIMType datatable, int fromrow, int torow) {
      int row, column; 
      int LineCount, dummy;                  /* Function displays list of every data value - not just latest 10.*/

      if (fromrow < 0) {  fromrow = 0;  }
      if (torow > datatable.currentrow-1) {  torow = datatable.currentrow-1;  }  /* Don’t display empty rows.*/
      if (datatable.currentrow == 0) {  printf("The current table is empty\n");  }
      else {
          LineCount=0;  
	      printf("        Time\t\t Air Density\t\t        Drag\t\tAcceleration\t\t    Velocity\t\t    Altitude\t\t     Gravity\t\tMassofRocket\n"); 		        /* Column headings.*/              
          for (row=fromrow;  row<=torow;  row=row+1) {
	        
	           LineCount = LineCount+1;
               if (LineCount==10) {
                    printf("Press RETURN to continue ...\n"); 
                    fflush(stdin);
                    scanf("%c",&dummy);
                    LineCount=0;}
               for (column=1;  column<=MAXCOL;  column=column+1) {
		            printf("%9.2f\t\t", datatable.table[row][column]);     
	           } 
	           printf("\n");
          }                   
      }
}

RSIMType ClearDataTable(RSIMType datatable) {       /* This function clears the entire current set of data.*/
     char confirmclear;
     int  row, column;
     confirmclear = 'D';                        /* Causes the program to enter the while loop below.*/
     
     if (datatable.currentrow == 0) {
         confirmclear = ' '; }
	 while (confirmclear != 'Y' && confirmclear != 'y' && confirmclear != 'N' && confirmclear != 'n') {  /* Allows for both uppercase and lowercase.*/
	        printf("The current table contains data which will be lost. Are you sure you want to clear the current table, (Y/N)?");
	        scanf(" %c", &confirmclear);       /* Asks the user if they are sure they want to clear the table.*/
            printf("\n"); }
         if (confirmclear == 'Y' || confirmclear == 'y') {  /* If the choice is yes, then all the row's values are set to 0.*/
	         for (row=0;  row<=MAXROW-1;  row=row+1) {      /* Otherwise returns to the menu.*/
		          for (column=1;  column<=MAXCOL;  column=column+1) {
		               datatable.table[row][column] = 0; }
	         }
         datatable.currentrow = 0; }  /* Finally the current row's data is set to 0.*/
      return datatable; 
}



float max_function(RSIMType datatable, int column){
    float max;
    int row;
    /*printf("max start 0:0 %f\n",datatable.table[0][column]);*/
    max = datatable.table[0][column];
    for (row=0;  row<MAXROW;  row=row+1){ 
        if( datatable.table[row][column] > max ) {
            max = datatable.table[row][column];
        }
    }
    return max;
    
} 
      
/* A sclaing function where x is the x data point from the initial equations, xres is the bottom right x coordinate of the graph you wish
 to plot on and xmax is the maximum x value in the data set*/      

int scale(int start, int end, RSIMType datatable, int row, int column, int max){ 
      return start + (end - start) * datatable.table[row][column] / max;
}

  
 void plot_graph(int column_x, int column_y, RSIMType datatable, int xs, int ys, int xe, int ye, int color){
     float xmax, ymax;     
     int row;
     int x1,x2,y1,y2;
     /*printf("plot_graph start colX:%d\n",column_x);*/
     xmax = max_function(datatable, column_x);
     ymax = max_function(datatable, column_y);
     for (row=0;  row<MAXROW-1;  row=row+1){
        /*x1 = (xborderless/MAXROW)*row;*/
        x1 = scale(xs, xe, datatable, row, column_x, xmax);
        y1 = scale(ys, ye, datatable, row, column_y, ymax);
        x2 = scale(xs, xe, datatable, row+1, column_x, xmax);
        y2 = scale(ys, ye, datatable, row+1, column_y, ymax);
        /*scale(xborderless, xmax, datatable, row, column_x, x_origin);*/
        GrLine(x1, y1, x2, y2, color);
     }             
}
      
void Graph_plotter(int column_x, int column_y, RSIMType datatable,  int max, float xborderless, float yborderless) {
    int xres, yres, ob, ib;
    int x1a, y1a, x2a, y2a, x1b, y1b, x2b, y2b;
    char str[80];
    char temp[80];
    GrMouseEvent evt;
    GrSetMode(GR_width_height_graphics,1024,600);
    ob = 40;
    ib = 10;   
    xres=GrScreenX();
    yres=GrScreenY();
    xborderless = (xres - (ob/2));
    yborderless = (yres/2)+(ob/2)-(2*(ib));                                 /* plotting graphs, axes and labels*/
    
    x1a = ob/2;
    y1a = (yres/2)+(ob/2)-2*(ib);
    x2a = xres-(ob/2);
    y2a = (ob/2)+ib;
    
    x1b = ob/2;
    y1b = yres-(3*ib);
    x2b = (2*(xres-ob))/3;
    y2b = ((yres+20)/2 + 2*ib);

    GrLine(x1a, y1a, x1a, y2a, 15);
    GrLine(x1a, y1a, x2a, y1a, 15);
    GrLine(x2a, y1a, x2a, y2a, 15);
    GrLine(x1b, y1b, x1b, y2b, 15);
    GrLine(x1b, y1b, x2b, y1b, 15);
    DrawText1(xres/2, 0, displaytitle, GR_ALIGN_CENTER, GR_ALIGN_TOP );    /*labels*/
    DrawText2(ib, (yres/4)+((3/2)*ib), graph1yalabel, GR_ALIGN_CENTER, GR_ALIGN_CENTER );
    DrawText2(xres - ib, (yres/4)+((3/2)*ib), graph1yblabel, GR_ALIGN_CENTER, GR_ALIGN_CENTER );
    DrawText1(xres/2, ((yres/2)+(ib)), graph1xlabel, GR_ALIGN_CENTER, GR_ALIGN_TOP );
    DrawText2(ib, yres-(yres/4), graph2ylabel, GR_ALIGN_CENTER, GR_ALIGN_CENTER );
    DrawText1(xres/3, yres-2*ib, graph2xlabel, GR_ALIGN_CENTER, GR_ALIGN_TOP );
    
    GrFilledBox(((2*(xres-ob))/3)+(ob/2),(yres/2)+(ob/2)+ib,xres-(ob/2),yres-(ob/2),8);
    DrawText1(((2*(xres-ob))/3)+(ob/2)+ib,(yres/2)+(ob), graph1yalabel, GR_ALIGN_LEFT, GR_ALIGN_TOP);
    DrawText1(((2*(xres-ob))/3)+(ob/2)+ib,(yres/2)+(2*ob), graph2ylabel, GR_ALIGN_LEFT, GR_ALIGN_TOP);
    DrawText1(((2*(xres-ob))/3)+(ob/2)+ib,(yres/2)+(3*ob), graph2xlabel, GR_ALIGN_LEFT, GR_ALIGN_TOP);
    DrawText1(((2*(xres-ob))/3)+(ob/2)+ib,(yres/2)+(4*ob), graph1xlabel, GR_ALIGN_LEFT, GR_ALIGN_TOP);
    
    plot_graph(1, 5, datatable, x1a, y1a, x2a, y2a, 15); /* function to plot velocity against time on the top graph*/
    plot_graph(1, 4, datatable, x1a, y1a, x2a, y2a, 4); /* function to plot accleration against time on the top graph*/
    plot_graph(1, 6, datatable, x1b, y1b, x2b, y2b, 5); /* function to plot altitude against time on the top graph*/
    
    /* GrContext *GrCurrentContext(void) or GetScreenContext()         */
    while(1){

        GrMouseGetEventT(GR_M_EVENT,&evt,0L);
        
        if(evt.flags & (GR_M_KEYPRESS | GR_M_BUTTON_CHANGE))
        {
            
            sprintf(str, "GrScreenX %d GrScreenY %d", GrScreenX(), GrScreenY());
            
            DrawText1(100,100,str,GR_ALIGN_CENTER, GR_ALIGN_CENTER);
        }
        
    }
    
}




int DrawText1(int x, int y, char * message, int xAlign, int yAlign) {          /*horizontal axes, labels*/                                                
    GrTextOption grt;                                                     
    grt.txo_font = &GrDefaultFont;                                        
    grt.txo_fgcolor.v = GrWhite();
    grt.txo_bgcolor.v = GrNOCOLOR;
    grt.txo_direct = GR_TEXT_RIGHT;
    grt.txo_xalign = xAlign;
    grt.txo_yalign = yAlign;
    grt.txo_chrtype = GR_BYTE_TEXT;
    GrDrawString( message,strlen( message ),x,y,&grt ); 
}     
    
    
    
    
int DrawText2(int x, int y, char * message, int xAlign, int yAlign) {             /*vertical axes, labels*/                                             
    GrTextOption grt;                                                     
    grt.txo_font = &GrDefaultFont;                                        
    grt.txo_fgcolor.v = GrWhite();
    grt.txo_bgcolor.v = GrNOCOLOR;
    grt.txo_direct = GR_TEXT_UP;
    grt.txo_xalign = xAlign;
    grt.txo_yalign = yAlign;
    grt.txo_chrtype = GR_BYTE_TEXT;
    GrDrawString( message,strlen( message ),x,y,&grt ); 
}     


float timer(float dt){
      time = time + dt;
      return time;
}   


float calc_density(float air_temp, float molar_mass) {
      float pressure = pressure_at_sea_level * pow(exp(1),(-1 * molar_mass * gravity * altitude) / (gas_constant * air_temp));
      density = (pressure * molar_mass)/(gas_constant * air_temp);
      return density;
      }
/*for temperature, i have googled rocket launches from the uk. on of the sites i a place called south uist. I have found the average temperature there to be about 7 degrees C so we could use this as our default case and then on the start menu when it says what the programme is we could just say "this programme is designed to simulate an Atlas V launching from Soutch Uist" or something like that*/ 

float calc_drag(float drag_coefficient, float area_which_experiences_drag) {
      drag = (drag_coefficient * density * pow(velocity, 2) * area_which_experiences_drag) / 2;
      return drag;
      }

float calc_acceleration(float thrust) {
      acceleration = ((thrust - drag) / mass) - gravity;
      return acceleration;
      }                          
           
float calc_velocity(float dt) {
     velocity = velocity + (acceleration * dt);
     
     return velocity;
     }

float calc_altitude(float dt) {
      altitude = altitude + velocity * dt + ((acceleration * (dt * dt)) / 2);
      return altitude;
      }

float calc_gravity() {
      gravity = (gravitational_constant * mass_of_earth) / (pow(radius_of_earth + altitude,2));
      return gravity;
      }

float calc_mass(float fuel_rate, float dt) {
      mass = mass - fuel_rate * dt;
      return mass;
      }


RSIMType AddData(RSIMType datatable) {
     float total_time = 250;     /*changeable?*/     
     float drag_coefficient = 0.42;
     float area_which_experiences_drag = (PI / 4) * pow(5.4, 2);
     /*float density;
     float drag; 
     float acceleration;*/
     float molar_mass = 0.02897;
     float fuel_rate = 450; /*need to look this up*/
     float thrust = 8991400; /*made up*/
     float air_temp = 280; /*made up*/
     float dt = 1.0; /*let them choose this in proper one*/
     velocity = 0;
     altitude = 10;  
     mass = 500000; /*this is madeup*/
     gravity = 9.7984;
     time = 0;
     
           /* Function takes data input and stores it in the data structure, row by row.*/
     if (datatable.currentrow >= (MAXROW)) {
    	   printf("The array of data is full"); }
     /*else if (time = total_time) {
           printf("The array of data is full"); } /* Displayed if array is full.*/
     else {
           while (time < total_time) { 
                    datatable.table[datatable.currentrow][COLt] = timer(dt);  
	                datatable.table[datatable.currentrow][COLRo] = calc_density(air_temp, molar_mass);  
	                datatable.table[datatable.currentrow][COLdrag] = calc_drag(drag_coefficient,area_which_experiences_drag);
	                datatable.table[datatable.currentrow][COLa] = calc_acceleration(thrust);
	                datatable.table[datatable.currentrow][COLv] = calc_velocity(dt);
	                datatable.table[datatable.currentrow][COLh] = calc_altitude(dt);
	                datatable.table[datatable.currentrow][COLg] = calc_gravity();
	                datatable.table[datatable.currentrow][COLm] = calc_mass(fuel_rate, dt);
	                datatable.currentrow = datatable.currentrow + 1;
                    }                                               
              return datatable; }
    }   
      
           
      



int ChangeParameters(int payload, int number_of_boosters, int inert_mass, int centaur_engine_type, int i){
    int  inputfinished;
    char userinput;
    inputfinished = 0;
	while (inputfinished == 0) {
          printf("\nP=Mass of Payload  B=Number of Boosters  I=Inert Mass   C=Centaur Engine Type   T= Temperature   X=Return to Menu\n");
		  scanf(" %c", &userinput);

          switch(toupper(userinput)) {                              /* 'toupper' converts all inputs to uppercase - lowercase can then be used too - user's choice.*/
                 case 'P': MassOfPayload(payload, i); break;           /* Case statement with the different options to the user.*/
                 case 'B': NumberOfBoosters(number_of_boosters); break;                                                       
                 case 'I': InertMass(inert_mass); break;
                 case 'C': CentaurEngineType(centaur_engine_type, i); break;           
                 case 'T': printf("temp function"); break;
                 case 'X': inputfinished = 1; break;
                 default: printf("\nInvalid option, try again\n"); }     
    } 
    return payload, number_of_boosters, inert_mass, centaur_engine_type; 
}      
    

int MassOfPayload(int payload, int i) {
      printf ("Would you like to have:\n1. Short pay load\n2. Medium pay load\n3. Long pay load\n");
      i = ValidateData();
      if (i == 1){ 
            payload = 3524;}
      else if (i == 2){
            payload = 4003;} 
      else if (i == 3){ 
            payload = 4379;}
      else { printf("\nInvalid entry, please try again\n"); }  
      return payload;
      }

int NumberOfBoosters(int number_of_boosters) {
      printf ("How many boosters would you like? (You can have 0-5): ");
      number_of_boosters = ValidateData();
      return number_of_boosters;            /*how to prevent incorrect answers?*/
      }

int InertMass(int inert_mass, int i) {
      printf ("What type of JOECOLEBROOK would you like to use?:\n1. 5x1\n2. 5x2\n");
      i = ValidateData();
      if (i == 1) { inert_mass = 4459; } /*common centaur + interstage adapter*/
      else if (i == 2) { inert_mass = 4689; }/*common centaur + interstage adapter*/
      else { printf("\nInvalid entry, please try again\n"); }
      return inert_mass;
      }
      

/*float calc_mass(int mass, int payload, int number_of_boosters, int inert_mass); {
      mass = payload + number_of_boosters * 46697 + inert_mass + 275 + 20830 + 285 + 284089 + 21351; /*payload, boosters, inert mass, common centaur(CFLR, Propellant), Atls booster cylindrical interstage adapter, atlas booster(propellant, inert mass*/
    /*  return mass;
      }*/

int CentaurEngineType(int centaur_engine_type, int i) {
      printf ("What type of centaur engine would you like to use:\n1. Single engine centaur\n2. Duel engine centaur\n");
      i = ValidateData();
      if (i = 1) {centaur_engine_type = 99200; }
      else if (i = 2) {centaur_engine_type = 198400; }
      else { printf("\nInvalid entry, please try again\n"); }
      return centaur_engine_type;
      }

/*float thrust_calc(int thrust, int centaur_engine_type, int i) {
      thrust = centaur_engine_type + (number_of_boosters * 1688400) + 3827000;
      return thrust;
      }*/



int main() {                                                            /* Main function.*/
    RSIMType datatable;
    int choice;
    int payload;
    int number_of_boosters; 
    int inert_mass;   
    int centaur_engine_type;
    int i = 0;
    int max = 0;      
    int xborderless;
    int yborderless;                                          /* Variables declared, and current row in the data table set to 0.*/
    datatable.currentrow = 0;
    choice = 99;                                                        /* Enters the while loop.*/
    while(choice != 0) {
        choice = Menu();                         /* Case statements corresponding to the users choice.*/
        switch(choice) {                                                /* Each one calls a function to perform a certain task.*/
                case 1:  datatable = AddData(datatable);break;          /* If the input is incorrect, an error message will be displayed.*/
                case 2:  DisplayDataTable(datatable, 0, MAXROW); break;                                               /* When a function like this is called - the data structure is*/
                case 3:  datatable = ClearDataTable(datatable); break;
                case 4:  ChangeParameters(payload, number_of_boosters, inert_mass, centaur_engine_type, i); break;                             /* The break stops the while loop from running through each option*/                                                             
                case 5:  Graph_plotter(column_x, column_y, datatable, max, xborderless, yborderless); break;
                case 6:  choice = 0; break;                               /* once a case has been selected.*/   
                default: printf("\nInvalid entry, please try again\n"); } 
   }
   printf("Program ended.\n");
   return 0; }

