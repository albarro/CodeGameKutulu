using System;
using System.Linq;
using System.IO;
using System.Text;
using System.Collections;
using System.Collections.Generic;

//Minions states
enum STATES
{
    SPAWNING = 0,
    WANDERING = 1,
    STALKING = 2,
    RUSHING = 3,
    STUNNED = 4
}

/**
 * Survive the wrath of Kutulu
 * Coded fearlessly by JohnnyYuge & nmahoude (ok we might have been a bit scared by the old god...but don't say anything)
 **/
class Player
{
    static void Main(string[] args)
    {
        string[] inputs;
        int width = int.Parse(Console.ReadLine());
        int height = int.Parse(Console.ReadLine());
        string[] lines = new string[height];
        for (int i = 0; i < height; i++)
        {
            lines[i] = Console.ReadLine();
        }

        inputs = Console.ReadLine().Split(' ');
        int sanityLossLonely = int.Parse(inputs[0]); // how much sanity you lose every turn when alone, always 3 until wood 1
        int sanityLossGroup = int.Parse(inputs[1]); // how much sanity you lose every turn when near another player, always 1 until wood 1
        int wandererSpawnTime = int.Parse(inputs[2]); // how many turns the wanderer take to spawn, always 3 until wood 1
        int wandererLifeTime = int.Parse(inputs[3]); // how many turns the wanderer is on map after spawning, always 40 until wood 1

        int wardenerAttack = 20;

        //Load cell costs
        SortedDictionary<string, int> cellCosts = new SortedDictionary<string, int>();
        //Wall
        cellCosts.Add("#", int.MaxValue);
        //Spawn
        cellCosts.Add("w", wardenerAttack);
        //Empty cell
        cellCosts.Add(".", sanityLossLonely);
        //Shelter 
        cellCosts.Add("U", sanityLossLonely); // A shelter cell make us loss the same sanity as th rest
        //ENTITIES
        // WANDERER 
        cellCosts.Add("WA", 15);
        // SLASHER 
        cellCosts.Add("SL", 30);
        //EffectPlan
        cellCosts.Add("EP", -4);
        //EffectShelther
        cellCosts.Add("ES", -1);
        //Explorers
        cellCosts.Add("explorer", sanityLossGroup - sanityLossLonely);

        int planUses = 2; // defined by the rules
        int planDuration = 4;
        int planRange = 2;

        int sanityRecoverByPlan = 3;
        int sanityRecoverByExplorerOnPlan = 3;
        int remainingPlanTicks = 0;

        int lightUses = 3; // defined by the rules
        int lightRange = 5;
        int lightDuration = 2;
        int remaingLightTicks = 0;
        int startingSanity = 250;

        Explorer me = new Explorer(-1, true, startingSanity); // Our explorer


        // game loop
        while (true)
        {
            Map map = new Map(width, height, lines, cellCosts, sanityLossLonely);

            //map.printMapa();

            int entityCount = int.Parse(Console.ReadLine()); // the first given entity corresponds to your explorer

            for (int i = 0; i < entityCount; i++)
            {
                inputs = Console.ReadLine().Split(' ');
                string entityType = inputs[0];
                int id = int.Parse(inputs[1]);
                int x = int.Parse(inputs[2]);
                int y = int.Parse(inputs[3]);
                int param0 = int.Parse(inputs[4]);
                int param1 = int.Parse(inputs[5]);
                int param2 = int.Parse(inputs[6]);

                //Console.Error.WriteLine("i= " + i + " type= " + entityType + " | x=" + x + " y=" + y);

                switch (entityType)
                {
                    case "EXPLORER":
                        Explorer exp = new Explorer(id, i == 0, param0);
                        map.addExporer(exp, new TaxiPoint(x, y));
                        if (exp.IsMe)
                        {
                            me = exp;
                            //Console.Error.WriteLine("Me id:{0}", me.Id);
                        }
                        break;
                    case "WANDERER":
                        param0 = (param1 == 0) ? -param0 : param0;
                        map.addWanderer(new Wanderer(id, param2, param0), new TaxiPoint(x, y));
                        break;
                    case "SLASHER":
                        STATES state = (STATES)param1;
                        map.addSlasher(new Slasher(id, param2, state), new TaxiPoint(x, y));
                        break;
                    //This two are treated the same, but they shouldn't
                    case "EFFECT_PLAN":
                        map.addPlan(new Effect(id, param0), new TaxiPoint(x, y));
                        break;
                    case "EFFECT_SHELTER":
                        map.addShelter(new Effect(id, param0), new TaxiPoint(x, y));
                        break;
                    /*case "EFFECT_LIGHT":
                        map.addPlan(new Effect(id, param0), new TaxiPoint(x,y));
                        break;*/
                    default:
                        break;
                }

            }

            map.calculateCandidates();

            // Write an action using Console.WriteLine()
            // To debug: Console.Error.WriteLine("Debug messages...");

            //Console.WriteLine("WAIT"); // MOVE <x> <y> | WAIT

            if (remainingPlanTicks > 0)
            {
                remainingPlanTicks--;
            }

            //map.printMapa();

            TaxiPoint desiredPos = map.getBestCell();

            //map.printMapa();

            if (desiredPos is null)
            {
                wait();
            }
            else
            {
                //If sanity low
                if (me.Sanity < 100 && planUses > 0 && remainingPlanTicks < 1)
                {
                    //TODO
                    plan();
                    planUses--;
                    remainingPlanTicks = planDuration;
                }
                else
                {
                    move(desiredPos);
                }
            }


        }
    }

    static void move(TaxiPoint point)
    {
        Console.WriteLine("MOVE " + point.PosX + " " + point.PosY);
        Console.Error.WriteLine("MOVE " + point.PosX + " " + point.PosY);
    }

    static void wait()
    {
        Console.WriteLine("WAIT");
    }

    static void plan()
    {
        Console.WriteLine("PLAN");
    }

    static void light()
    {
        Console.WriteLine("LIGHT");
    }
}

class Map
{

    private int Width;
    private int Height;

    //Costs
    private SortedDictionary<string, int> CellCosts;
    int bestMaxCost = 2;

    private Cell[,] MapArray;
    private List<Cell> CellCandidates;

    private TaxiPoint PlayerPos;
    private int PlayerId;

    public Map(int width, int height, string[] staticMap, SortedDictionary<string, int> cellCosts, int minBestCell)
    {
        CellCandidates = new List<Cell>();
        bestMaxCost = minBestCell;

        Width = width;
        Height = height;

        MapArray = new Cell[width, height];
        CellCosts = cellCosts;

        for (int y = 0; y < height; y++)
        {

            string[] line = staticMap[y].Select(x => x.ToString()).ToArray();
            // foreach (var item in line)
            // {
            //     Console.Error.WriteLine(item);    
            // }


            for (int x = 0; x < width; x++)
            {
                Cell cell = new Cell(CellCosts[line[x]], line[x], new TaxiPoint(x, y));
                // Console.Error.WriteLine(cell);

                MapArray[x, y] = cell;
            }
        }

    }

    public void calculateCandidates()
    {

        for (int x = 0; x < Width; x++)
        {
            for (int y = 0; y < Height; y++)
            {
                if (MapArray[x, y].Cost < bestMaxCost)
                {
                    CellCandidates.Add(MapArray[x, y]);
                }
            }
        }

        CellCandidates.Sort();
    }

    public void changeCellCost(int newCost, TaxiPoint center)
    {
        MapArray[center.PosX, center.PosY].Cost = newCost + MapArray[center.PosX, center.PosY].Cost;
    }

    public void changeCellCostInArea(int newCost, int range, TaxiPoint center)
    {
        int minX = (center.PosX - range < 0) ? 0 : center.PosX - range;
        int maxX = (center.PosX + range > Width) ? Width : center.PosX + range;

        int minY = (center.PosY - range < 0) ? 0 : center.PosY - range;
        int maxY = (center.PosY + range > Height) ? Height : center.PosY + range;

        for (int x = minX; x < maxX; x++)
        {
            for (int y = minY; y < maxY; y++)
            {
                if (MapArray[x, y].Cost < int.MaxValue)
                    MapArray[x, y].Cost += newCost;
            }
        }
    }

    // For simplicity only check x and y axis
    public void changeCellCostInLOS(int newCost, int range, TaxiPoint center)
    {
        int minX = (range == int.MaxValue || center.PosX - range < 0) ? 0 : center.PosX - range;
        int maxX = (range == int.MaxValue || center.PosX + range > Width) ? Width : center.PosX + range;

        int minY = (range == int.MaxValue || center.PosY - range < 0) ? 0 : center.PosY - range;
        int maxY = (range == int.MaxValue || center.PosY + range > Height) ? Height : center.PosY + range;

        //Console.Error.WriteLine("Center {0} {1}", center.PosX, center.PosY);
        //Console.Error.WriteLine("Upd cost minx:{0} masx:{1} miny:{2} maxy:{3}", minX, maxX, minY, maxY);
        //Check From center to min x
        for (int x = center.PosX; x > minX; x--)
        {
            if (MapArray[x, center.PosY].Cost < int.MaxValue)
                MapArray[x, center.PosY].Cost += newCost;
            else //Wall break LOS we stop
                break;
        }

        //Check From center to max x
        for (int x = center.PosX; x < maxX; x++)
        {
            if (MapArray[x, center.PosY].Cost < int.MaxValue)
                MapArray[x, center.PosY].Cost += newCost;
            else //Wall break LOS we stop
                break;
        }

        //Check From center to min y
        for (int y = center.PosY; y > minY; y--)
        {
            if (MapArray[center.PosX, y].Cost < int.MaxValue)
                MapArray[center.PosX, y].Cost += newCost;
            else //Wall break LOS we stop
                break;
        }

        //Check From center to max y
        for (int y = center.PosY; y < maxY; y++)
        {
            if (MapArray[center.PosX, y].Cost < int.MaxValue)
                MapArray[center.PosX, y].Cost += newCost;
            else //Wall break LOS we stop
                break;
        }
    }

    public void addExporer(Explorer exp, TaxiPoint point)
    {
        if (exp.IsMe)
        {
            PlayerPos = point;
            PlayerId = exp.Id;
            //Console.Error.WriteLine("Me id:{0}", exp.Id);
        }
        else if (exp.Active)
        {
            //changeCellCostInArea(CellCosts["explorer"], 2, point);
            int range = 2;
            //Explorer is a special case
        int minX = (point.PosX - range < 0) ? 0 : point.PosX - range;
        int maxX = (point.PosX + range > Width) ? Width : point.PosX + range;

        int minY = (point.PosY - range < 0) ? 0 : point.PosY - range;
        int maxY = (point.PosY + range > Height) ? Height : point.PosY + range;

        for (int x = minX; x < maxX; x++)
        {
            for (int y = minY; y < maxY; y++)
            {
                if (MapArray[x, y].Cost < int.MaxValue && MapArray[x, y].Entity is not Explorer)
                    MapArray[x, y].Cost += CellCosts["explorer"];
                    MapArray[x, y].Entity = exp;
            }
        }
        }
    }

    public void addPlan(Effect plan, TaxiPoint point)
    {
        if (plan.Active)
        {
            changeCellCostInLOS(CellCosts["EP"], 2, point);
        }
    }

    public void addShelter(Effect plan, TaxiPoint point)
    {
        if (plan.Active)
        {
            changeCellCost(CellCosts["ES"], point);
        }
    }

    public void addWanderer(Wanderer wand, TaxiPoint point)
    {
        if (wand.Target == PlayerId && wand.Active)
        {
            changeCellCostInLOS(CellCosts["WA"] + 5, 3, point);
        }
        else if (wand.Active)
        {
            changeCellCostInArea(CellCosts["WA"], 2, point);
        }
    }

    public void addSlasher(Slasher slasher, TaxiPoint point)
    {
        if (slasher.Target == PlayerId && slasher.Active)
        {
            changeCellCostInLOS(CellCosts["SL"] + 10, int.MaxValue, point);
            //Console.Error.WriteLine("Adding Slasher in {0} {1}", point.PosX, point.PosY);
        }
        else if (slasher.Active)
        {
            changeCellCostInLOS(CellCosts["SL"], int.MaxValue, point);
            //Console.Error.WriteLine("Adding Slasher in {0} {1}", point.PosX, point.PosY);
        }
    }

    public TaxiPoint getBestCell()
    {
        Cell actualCell = MapArray[PlayerPos.PosX, PlayerPos.PosY];
        
        foreach (var cell in CellCandidates)
        {
            //Console.Error.WriteLine("Cost Before:{0}",cell.Cost);
            cell.Cost = findShortPathCost(actualCell, cell);
            //Console.Error.WriteLine("Cost After:{0}",cell.Cost);
        }

        //we rearenge the candidates now with travel costs and get the best one
        CellCandidates.Sort();

        //printCandidatos();

        Cell bestCell = (CellCandidates.Any()) ? CellCandidates.First() : actualCell;
        //Console.Error.WriteLine(bestCell);

        return bestCell.Cord;
    }

    private int findShortPathCost(Cell startingCell, Cell destinationCell)
    {
        // applying BFS on matrix cells starting from source
        //Console.Error.WriteLine("Check path for {0}", destinationCell);
        Queue<Cell> queue = new Queue<Cell>();
        queue.Enqueue(startingCell);
        startingCell.Distance = startingCell.Cost;

        bool[,] visited = new bool[Width, Height];
        visited[startingCell.Cord.PosX, startingCell.Cord.PosY] = true;

        while (queue.Any())
        {
            Cell p = queue.Dequeue();
            int x = p.Cord.PosX;
            int y = p.Cord.PosY;
            //Console.Error.WriteLine("Check cell {0}", p);

            // Destination found
            if (p.Equals(destinationCell)){
                //Console.Error.WriteLine("Path Found Cost={0}",p.Distance);
                return p.Distance;
            }

            Cell nextCell;
            // rigth
            if (checkValidCell(x + 1, y, visited))
            {
                nextCell = MapArray[x+1, y];
                nextCell.Distance = p.Distance + nextCell.Cost;
                queue.Enqueue(nextCell);
                visited[x+1, y] = true;
            }

            // left
            if (checkValidCell(x - 1, y, visited))
            {
                nextCell = MapArray[x-1, y];
                nextCell.Distance = p.Distance + nextCell.Cost;
                queue.Enqueue(nextCell);
                visited[x-1, y] = true;
            }

            // up
            if (checkValidCell(x, y-1, visited))
            {
                nextCell = MapArray[x, y-1];
                nextCell.Distance = p.Distance + nextCell.Cost;
                queue.Enqueue(nextCell);
                visited[x, y-1] = true;
            }

            // down
            if (checkValidCell(x, y+1, visited))
            {
                nextCell = MapArray[x, y+1];
                nextCell.Distance = p.Distance + nextCell.Cost;
                queue.Enqueue(nextCell);
                visited[x, y+1] = true;
            }
        }
        //No path
        return int.MaxValue;
    }

private bool checkValidCell(int x, int y, bool[,] visited)
{
    //Check coords
    if (x > Width || x < 0 || y < 0 || y > Height)
    {
        return false;
    }
    //Check walls
    if (MapArray[x, y].isWall)
    {
        return false;
    }
    //Check if visited
    if (visited[x, y])
    {
        return false;
    }

    return true;
}


public void printMapa()
{
    Console.Error.Write("\t");
    for (int xIndex = 0; xIndex < Width; xIndex++)
    {
        Console.Error.Write("x" + xIndex + "\t");
    }
    Console.Error.WriteLine();


    for (int y = 0; y < Height; y++)
    {
        Console.Error.Write("y" + y + "\t");
        for (int x = 0; x < Width; x++)
        {
            if (MapArray[x, y].Cost == int.MaxValue)
                Console.Error.Write(MapArray[x, y].Label + "\t");
            else
                Console.Error.Write(MapArray[x, y].Cost + "\t");
        }
        Console.Error.WriteLine();
    }
}

public void printCandidatos()
{
    Console.Error.WriteLine("Candidatos: ");
    foreach (var cell in CellCandidates)
    {
        Console.Error.WriteLine(cell);
    }
}
}

// Representation of a map cell
class Cell : IComparable<Cell>
{

    public string Label;
    public int Cost { get; set; }
    public TaxiPoint Cord { get; }
    public Entity Entity { get; set; }
    public bool isWall{
        get => Label.Equals("#");
    }
    public int Distance {get; set;}

    public Cell(int cost, string label, TaxiPoint cord)
    {
        Cost = cost;
        Cord = cord;
        Label = label;
        Entity = null;
        Distance = int.MinValue;
    }

    public override string ToString()
    {
        StringBuilder res = new StringBuilder();
        res.AppendFormat("[{0}|x:{1} y{2}|Cost:{3}", Label, Cord.PosX, Cord.PosY, Cost);
        if(Entity is not null){
            res.AppendFormat("|Entity:{0}",Entity);
        }
        if(Distance > int.MinValue){
            res.AppendFormat("|Distance:{0}", Distance);
        }
        res.Append("]");
        return res.ToString();
    }

    // Equals if Cord are the same
    public bool Equals(Cell other)
    {

        return this.Cord.Equals(other.Cord);
    }

    // They are ordered by cost
    public int CompareTo(Cell other)
    {
        if (other == null)
            return 1;

        else
            return this.Cost.CompareTo(other.Cost);
    }

    //We have enough whit a shallowCopy, the Cords
    // shouldn't be modify even in copies of the cell
    public Cell ShallowCopy()
    {
        return (Cell)this.MemberwiseClone();
    }
}

// Representation of entity
abstract class Entity
{

    public int Id { get; }
    public bool Active { get; }

    public Entity(int id)
    {
        Id = id;
    }
}

class Explorer : Entity
{

    public bool IsMe { get; }
    public int Sanity { get; set; }
    public new bool Active
    {
        get => Sanity > 0;
    }

    public Explorer(int id, bool isMe, int sanity)
        : base(id)
    {
        IsMe = isMe;
        Sanity = sanity;
    }
}

class Wanderer : Entity
{

    public int Target { get; set; }
    public int TimeOfLive { get; set; } // Negative if not spawned; Positive if spawned
    public new bool Active
    {
        get => TimeOfLive > 0;
    }

    public Wanderer(int id, int target, int timeOfLive)
        : base(id)
    {
        Target = target;
        TimeOfLive = timeOfLive;
    }
}

class Slasher : Entity
{

    public int Target { get; set; }
    public STATES State { get; set; } // Negative if not spawned; Positive if spawned
    public new bool Active
    {// Active if hunting
        get => State == STATES.STALKING || State == STATES.RUSHING;
    }

    public Slasher(int id, int target, STATES state)
        : base(id)
    {
        Target = target;
        State = state;
    }
}


class Effect : Entity
{
    public int RemaingTicks { get; set; }
    public new bool Active
    {
        get => RemaingTicks > 0;
    }

    public Effect(int id, int remaingTicks)
        : base(id)
    {
        RemaingTicks = remaingTicks;
    }
}

class TaxiPoint
{

    public int PosX { get; }
    public int PosY { get; }

    public TaxiPoint(int x, int y)
    {
        PosX = x; PosY = y;
    }

    public double getDistanceToPoint(TaxiPoint otherPoint)
    {
        return Math.Abs(PosX - otherPoint.PosX) + Math.Abs(PosY - otherPoint.PosY);
    }

    public bool Equals(TaxiPoint other)
    {
        if (this.PosX == other.PosX && this.PosY == other.PosY)
        {
            return true;
        }
        else
            return false;
    }
}
